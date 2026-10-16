O tópico sobre resiliência, eu diria que é um dos mais importantes sobre arquitetura de microsserviços, pois precisamos, antes de tudo, partir do presuposto:
**"Em algum momento, todo sistema vai falhar..."**

E partindo dessa idéia, a pergunta que nos fazemos é:
- Como me preparar?
- Como mitigar os problemas que posso ter?
- Como ter o menor impacto possível

Aqui veremos as práticas utilizadas para mitigar esses riscos e tornar nossos microsserviços mais resiliente e toleráveis a falhas

## O que é resiliência?
Segundo o dicionário:
"Resiliência é um conjunto de estratégias adotadas ==**intencionalmente para a adaptação**== de um sistema quando uma falha ocorre"

E seguindo este ponto, precisamos ter estratégias de resiliência para nos possibilitar minimizar os riscos de perdas de dados e transações importantes para o negócio.
## Estratégias de resiliência
Primeiro de tudo que precisamos pensar, é que quando estamos a trabalhar com microsserviços é que sempre estaremos dentro de um ecossistemas enviando e recebendo requisições ou seja teremos dados para todo lado, com isso em mente, precisamos **nos proteger e ser protegidos**.

Um sistema de arquitetura distribuída precisa adotar mecanismos de autopreservação para 
garantir ao máximo sua operação com qualidade. E com qualidade, estamos nos referindo a excelência e garantia do mesmo resultado independente dos fatores que estão ocorrendo.

Um sistema não pode ser "teimoso" ao ponto de realizar mais requisições em um sistema que está falhando ou quebrado, por exemplo, se estamos vendo que um outro sistema no qual precisamos de um dado e este mesmo sistema está a retornar mensagens de timeout, porque continuar a enviar as mesmas requisições para este sistema? Este cenário é um que devemos ter em mente, para poder mitigar erros, proteger o nosso sistema e outros também.

Outro ponto crítico, é saber que um sistema lento no ar é pior do que um sistema fora do ar, pois com isso temos um efeito dominó, onde todos os outros sistemas que dependem deste sistema lento ficarão lentos também.

Aqui estão algumas das estrátegias que nos ajudam a tornar nosso sistema mais resiliente e confiável:
- [[Retry|Políticas de Retry]];
- [[Circuit Breaker]];
- [[Health checks]];
- [[Rate Limiting]];
- [[Outbox Pattern]]
- [[Idempotência e políticas de fallback]];
- [[Observabilidade/Introdução|Observabilidade]];
- [[Service Mesh]]