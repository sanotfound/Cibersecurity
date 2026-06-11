📂 Caso de ataque de DoS por SYN flood.
Um ataque DoS do tipo SYN Flood se caracteriza pela quantidade anormal de pacotes SYN que um servidor recebe, de forma que ele se sobrecarregue por estar recebendo mais pacotes SYN do que normalmente suporta, tornando-o indisponível.

📖 Contexto

Um servidor web começou a apresentar erro "connection timeout".
Os usuários não conseguiam acessar o serviço.


🔍 Análise e Evidências

Quando os visitantes do site tentam estabelecer uma conexão com o servidor web, ocorre um three-way handshake utilizando o protocolo TCP. 

Cliente envia solicitação de conexão com o servidor :
   “42584->443 [SYN] Seq=0 Win-5792 Len=120…”

2.  Servidor recebe a solicitação de conexão e retorna confirmação de conexão
   “443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120…”

3. Cliente recebe a confirmação de conexão e obtém sucesso ao se conectar com o serviço usando o protocolo TCP:
    “42584->443 [ACK] Seq=1 Win-5792 Len=120…”

Portanto, o agente-mal intencionado se aproveitou da seguinte forma:
1. Ao agente-mal intencionado enviar pacote SYN, o servidor recebe esse pacote
2. O servidor responde ao SYN com um SYN-ACK e reserva recursos aguardando o ACK final do cliente. ,mas conforme mais solicitações SYN aparecem, mais o servidor reserva esses recursos e mais ele fica sobrecarregado.
3. O ACK nunca chega porque o atacante não pretende concluir a conexão, e dessa forma, o servidor fica sobrecarregado pela quantidade de recursos que ele está reservando para cada solicitação SYN.
4. Se o ACK nunca chega, a conexão fica semi-aberta
E a partir disso, o servidor mantém milhares de conexões semi-abertas e esgotando seus recursos. Consequentemente, fazendo com que o servidor entre mau funcionamento e se torne inevitavelmente indisponível para qualquer pessoa. Qualquer funcionário que tentar acessar durante esse colapso não terá sucesso.


💡 Hipóteses

Os logs indicam que, conforme este agente foi mandando mais e mais pacotes SYN, o servidor foi ficando sobrecarregado. A partir desta linha acima, a inundação SYN ficou ainda mais evidente, pois este IP de origem (que é diferente da rede local dos funcionários) manda vários pacotes SYN sucessivos para o servidor, sobrecarregando-o e consequentemente tornando o serviço indisponível pela falta de conexão com o servidor, como mostra requisições seguintes por funcionários legítimos:

73 | 6.230548 | 192.0.2.1 | 198.51.100.16 | TCP | 443->32641 [RST, ACK] Seq=0 Win-5792 Len=120...
103 | 17.429678 | 192.0.2.1 | 203.0.113.0 | TCP | 443->54770 [RST, ACK] Seq=1 Win=5792 Len=0...
109 | 17.567768 | 192.0.2.1 | 203.0.113.0 | TCP | 443->54770 [RST, ACK] Seq=1 Win=5792 Len=0...


🛡️ Mitigações
Uma solução de bloqueio de IP não durará muito, pois um ataque deste tipo pode espalhar outros endereços IP para contornar esse bloqueio. 
Além disso, métodos de criptografia TLS (como o HTTPS) não funcionaria, já que a conexão TCP precisa ser estabelecida antes de qualquer coisa, e o ataque acontece justamente durante a fase three-way handshake do protocolo TCP.
A VPN não seria a melhor maneira pois só funcionaria em um cenário onde o servidor é acessível apenas por VPN, e dessa forma, o agente-mal intencionado não consegue atacar diretamente o servidor web, e para isso, ele precisaria se infiltrar e se autenticar na VPN para poder fazer isso, mas mesmo assim, a VPN reduz significativamente a superfície de ataque.

Uma boa forma de proteger a rede, impedindo que ataques desse tipo aconteçam novamente, é a implementação de um firewall de próxima geração (NGFW), pois ele pode ajudar a detectar padrões anormais de tráfego, aplicar políticas de filtragem e limitar conexões suspeitas, reduzindo os impactos de ataques de inundação SYN. Se for feita a implementação de um NGFW e o servidor acessível apenas por VPN, pode de fato realmente evitar este ataque ou diminuir drasticamente a superfície de ataque, já que estas duas ferramentas em conjunto pode elevar a postura de segurança dessa organização ainda mais.


