É uma ferramenta de mensageria, que usa o protocolo **AMQP 0-9-1** (**Advanced Message Queuing Protocol**), que é bastante usada para microserviços, parecidamente igual ao Apache Kafka. Basicamente, o RabbitMQ trabalha como um **Broker**.

![[Untitled.png]]

Ele contém um fila que os microserviços produtores (**publishers**) possam adicionar novas mensagens que os microserviços consumidores (**subscribers**) possam consumir. De forma geral, os produtores não conversam diretamente com as filas, eles mandam uma mensagem para um **Exchange** que tem o papel de colocar na fila correta a mensagem enviada, e com isso temos consumidores que estão conectados a diferentes filas e podem consumir as mensagens que estão nessas filas, como no exemplo abaixo:

![[f4210448-1f34-4bb8-9666-1152737b0528.png]]

Exchanges são entidades dentro do AMQP 0-9-1 onde as mensagens são enviadas. Elas levam a mensagem e roteiam para zero ou mais filas. O algoritmo usado depende do tipo de Exchange configurado. Existem 4 tipos de Exchange:

| **Exchange Type** | **Nome padrão pré-declarados** |
| --- | --- |
| Direct Exchange | (String vazia) e *amq.direct* |
| Fanout Exchange | *amq.fanout* |
| Topic Exchange | *amq.topic* |
| Headers Exchange | *amq.match* (and *amq.headers* in RabbitMQ) |

# Explicação do FullCycle

O RabbitMQ é um Service Broker, ou seja, um intermediário entre aplicações que conversam entre mensagens.

O RabbitMQ implementa diversos protocolos, incluindo:

- AMQP - Protocolo mais utilizado e baseado no TCP;
- MQTT;
- STOMP;
- HTTP

O RabbitMQ é desenvolvido em ERLANG, com isso ele demonstra uma rapidez enorme. Também tem um grande desacoplamento de serviços, tanto que dentro dele conseguimos instalar plugins e extensões, mas ao mesmo tempo para os microserviços a forma de comunicação entre eles é o próprio Service Broker.

## Por baixo dos panos…

![[Software Architecture/Attachments/image 3.png]]

Imaginemos que temos um client e um server, que é o item que fará conexão com o RabbitMQ, e essa conexão será feita via TCP, que é a base de todos os protocolos do RabbitMQ, e com sabemos quando trabalhamos com TCP, a primeira conexão que fazemos é um pouco lenta por conta de todo o processo de handshaking do TCP e assim a conexão acontece, sendo assim para evitar de toda a vez que abrirmos uma conexão para o Service Broker e ter essa lentidão, o RabbitMQ trabalha diferente, ele abre apenas uma conexão com o server, independente do número de Consumers, pois dentro de cada conexão ele cria “subconexões” que chamamos de “channels”, que trabalham da seguinte forma, os channels passam “por dentro” da conexão e farão toda a comunicação acontecer, e todo esse processo é chamado de “Multiplexing Connections”, que é onde abrimos diversas conexões dentro de uma outra ja existente, evitando todo o processo de inicialização. E toda a vez que abrimos uma channel, abrimos 1 thread dentro do servidor.

### Funcionamento Básico

![[image 1 1.png]]

- Publisher → É o ator que vai publicar a mensagem para o Consumer;
- Consumer → É o ator que lê a mensagem do Publisher;
- Queue → É a fila em si, onde o Publisher publicará suas mensagens e não de forma direta com o Consumer, que estará sempre “ouvindo” essa fila para ler as mensagens e depois processá-las;
- Exchange → É a responsável por checar cada mensagem enviada por um Publisher e envia-la na Queue correta, pois podemos ter diversas filas dentro do nosso serviço

### Tipos de Exchange

- Direct → É a exchange que manda a mensagem diretamente para uma determinada fila;
- Fanout → É uma exchange que manda uma mensagem para todas as filas que estão ”bindadas”, ou seja, que estão relacionadas com essa exchange. Por exemplo: se temos 10 filas estão relacionadas com uma exchange, quando mandamos uma mensagem, as 10 filas receberão aquela mensagem;
- Topic → É uma exchange que conseguimos colocar algumas regras, que dependendo da mensagem (ou como chamados “**routing key**”), ela vai encaminhar com de acordar com a regra, na fila que queremos
- Headers → É uma exchange que determinamos qual fila a mensagem será enviada pelos headers da mensagem

**Ilustração de uma Direct Exchange:**

![[image 2 1.png]]

**Ilustração de uma Fanout Exchange:**

![[image 3 1.png]]

**Ilustração de uma Topic Exchange:**

![[Software Architecture/Attachments/image 4.png]]

### Queues

As filas trabalham em seu padrão, que é o padrão FIFO (First In, First Out), ou seja primeira mensagem que entra, é a primeira que sai. Nas filas temos algumas propriedades:

- Durable: Se ela deve ser salva mesmo depois do restart do broker;
- Auto-Delete: A fila é removida automaticamente quando um consumer se desconecta da mesma;
- Expiry: Define o tempo de espera que uma fila ficará osciosa;
- Message TTL (Time-to-Live): Tempo de vida da mensagem, ou seja, quanto tempo essa mensagem ficará na fila, se caso ninguém consumir essa mensagem, ela será removida automaticamente;
- Overflow: Quando a fila transborda, ou seja, alcança seu máximo de mensagens, com isso temos algumas propriedades dentro dessa prorpriedade:
    - Drop head: Remove a última (mais antiga) mensagem e entra uma nova
    - Reject publish: Aqui o Publisher receberá uma mensagem dizendo que a mensagem foi rejeitada
- Exclusive: Somente o channel que criou pode acessar
- Max length ou bytes: Quantidade de mensagens que pode cair em uma fila, ou o tamanho máximo permitido da mensagem em bytes. Se uma mensagem cair em algum dos casos assim, teremos que agir com alguma das propriedades do Overflow

### Dead Letter Queues

Algumas mensagens não conseguem ser entregues por qualquer motivo, seja que quando ela caiu em uma fila, nenhum Consumer leu aquela fila, ou ela ficoui lá por muito tempo, etc, tendo isso em mente, encaminhamos essas mensagens para uma Exchange específica, baseada em uma configuração, que vai rotear essas mensagens para uma Dead Letter Queue. Dessa forma, essas mensagens serão consumidas por uma aplicação em comum e serão averiguadas posteriormente.

### Lazy Queues

São filas que armazenam as mensagens em disco, em alguns casos o fluxo, ou seja, a quantidade de mensagens para um Consumer que ele não consegue dar conta de ler essas mensagens com rapidez, e o RabbitMQ trabalha com memória e essa memória tem um limite, assim que esse limite vai chegando, precisamos garantir que os Consumers vão lendo as mensagens rapidamente, com isso as Lazy Queues pegam essas mensagens que não foram lidas rapidamente e guardam elas em disco para garantir que o Consumer mantenha seu ritmo e essas mensagens não são perdidade caso o Broker restarte.

Porém temos um problema grande aqui, esse tipo de estratégia exige um alto I/O, ou seja, acesso a um recurso direto da maquina, pois nesse caso estaremos acessando diretamente o disco do servidor. **Temos que pensar muito bem para o uso dessas filas.**

<aside>
☝🏽

**Simulador do RabbitMQ:** [tryrabbitmq.com](http://tryrabbitmq.com)

</aside>

## Confiabilidade

Comecemos com duas perguntas?

- Como garantir que as mensagens não serão perdidas no meio do caminho?
- Como garantir que as mensagens puderam ser processadas corretamente pelo consumers?

Pensando nessas perguntas, o RabbitMQ tem 3 recursos principais para resolver esses problemas:

- Consumer acknowledgement;
- Publisher confirm;
- Fila e mensagens são duráveis / persistidas

### Consumer Acknowledgement

Esse recurso trabalha da forma de quando um consumer receber uma mensagem, ele “alerta” que ele deu tudo certo.

Esse recurso tem 3 tipos:

- Basic.Ack;
- Basic.Reject;
- Basic.Nack;

**Basic.Ack**

Esse recurso é toda vez que mandamos uma mensagem para um consumer e o mesmo da um “ack” (acknowledgement), ou seja, ele response que esta tudo OK, ele recebeu e processou a mensagem enviada.

**Basic.Reject**

Esse recurso é total contrário do “Ack”, caso tenha tido qualquer problema na mensagem, processamento, exception na aplicação, etc, ele irá retornar um “reject” e com isso ele faz a mensagem voltar e ficar na fila novamente.

**Basic.Nack**

Esse recurso é parecido com o “Reject”, porém ele consegue rejeitar mais de uma mensagem ao mesmo tempo, diferente do “Reject” que só consegue rejeitar uma de cada vez.

### Publisher Confirms

Aqui fazemos a confirmação da mensagem no começo do processo, ou seja, o Publisher manda uma mensagem para o Exchange, e essa mensagem terá um ID, e através dele que iremos nos organizar para receber a confirmação. Esse ID deve ser um inteiro e é a Publisher que deve gerar esse ID manualmente.

Obs: Se caso, ao enviarmos uma mensagem e ela der problema, saibamos que esse problema é algo dentro do Exchange, e não no restante do fluxo do Broker. Nesse caso, iremos receber essa mensagem como um “Nack”.