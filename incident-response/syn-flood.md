# 🛡️ SYN Flood – Análise de um incidente de indisponibilidade

> **Objetivo:** Analisar um cenário de indisponibilidade de um servidor web e identificar, por meio da análise dos registros de rede, se o incidente é compatível com um ataque DoS do tipo SYN Flood.

---

# 📖 Contexto

Um servidor web começou a apresentar erro **"connection timeout"**, impedindo que os usuários acessassem o serviço.

Durante a investigação, foi observado um grande volume de solicitações TCP SYN destinadas ao servidor. O objetivo desta análise é compreender como esse comportamento pode levar à indisponibilidade do serviço.

---

# 🛠️ Tecnologias e conceitos envolvidos

- TCP
- Three-Way Handshake
- Protocolo HTTP
- Ataque DoS (Denial of Service)
- Análise de logs de rede

---

# 🔍 Análise

Quando um visitante tenta acessar um servidor web utilizando o protocolo TCP, ocorre o processo conhecido como **Three-Way Handshake**.

### 1. Cliente solicita uma conexão

```text
42584 -> 443 [SYN] Seq=0 Win=5792 Len=120
```

O cliente envia um pacote **SYN**, solicitando o estabelecimento de uma conexão.

### 2. Servidor responde à solicitação

```text
443 -> 42584 [SYN, ACK] Seq=0 Win=5792 Len=120
```

O servidor recebe o SYN e responde com um pacote **SYN-ACK**, reservando recursos internos enquanto aguarda a confirmação do cliente.

### 3. Cliente confirma a conexão

```text
42584 -> 443 [ACK] Seq=1 Win=5792 Len=120
```

Após receber o ACK, a conexão é estabelecida e a comunicação pode continuar normalmente.

---

# 📋 Como ocorre o ataque

Um ataque SYN Flood explora exatamente essa etapa do protocolo TCP.

O agente mal-intencionado envia milhares de pacotes SYN para o servidor.

Para cada pacote recebido:

1. O servidor recebe o SYN.
2. Responde com um SYN-ACK.
3. Reserva recursos internos aguardando o ACK final.

Entretanto, o atacante **nunca envia esse ACK**.

Consequentemente:

- milhares de conexões permanecem semi-abertas;
- o servidor continua reservando recursos para conexões que nunca serão concluídas;
- novos usuários legítimos deixam de ser atendidos;
- o serviço torna-se indisponível.

---

# 📋 Evidências

Durante a análise dos registros foi possível observar diversas tentativas de conexão fracassadas provenientes de usuários legítimos.

```text
73  | 6.230548  | 192.0.2.1 | 198.51.100.16 | TCP | 443->32641 [RST, ACK]

103 | 17.429678 | 192.0.2.1 | 203.0.113.0   | TCP | 443->54770 [RST, ACK]

109 | 17.567768 | 192.0.2.1 | 203.0.113.0   | TCP | 443->54770 [RST, ACK]
```

Esses registros demonstram que o servidor já não consegue estabelecer novas conexões normalmente, retornando pacotes **RST, ACK** para clientes legítimos.

Embora um pacote **RST, ACK** isoladamente não seja evidência suficiente de um ataque SYN Flood, neste contexto ele reforça a hipótese de que o servidor encontra-se sobrecarregado e incapaz de aceitar novas conexões.

---

# 💡 Hipótese

Com base na análise dos registros, o comportamento observado é compatível com um ataque **DoS do tipo SYN Flood**.

O grande volume de solicitações SYN mantém diversas conexões semi-abertas, consumindo recursos do servidor até que este deixe de responder às novas conexões de usuários legítimos, gerando erros como **connection timeout**.

---

# 🛡️ Mitigações

Bloquear apenas um endereço IP não é uma solução definitiva, pois ataques SYN Flood podem utilizar diversos endereços IP (ou endereços falsificados) para contornar esse bloqueio.

Além disso, mecanismos como **TLS/HTTPS** não impedem esse tipo de ataque, pois a negociação criptográfica ocorre somente após o estabelecimento da conexão TCP, enquanto o ataque acontece justamente durante o Three-Way Handshake.

O uso de uma **VPN** também não elimina completamente o problema, mas pode reduzir significativamente a superfície de ataque quando o serviço é acessível apenas por clientes autenticados.

Uma abordagem mais eficaz inclui:

- Implementação de um **Next Generation Firewall (NGFW)**;
- Rate Limiting para conexões TCP;
- SYN Cookies;
- IDS/IPS para detecção de padrões anormais;
- Monitoramento contínuo do tráfego de rede.

Essas medidas ajudam a reduzir o impacto de ataques SYN Flood e aumentam a disponibilidade dos serviços.

---

> **Observação:** Este estudo foi desenvolvido para fins educacionais, utilizando um cenário controlado e registros de rede disponibilizados em laboratório de cibersegurança.
