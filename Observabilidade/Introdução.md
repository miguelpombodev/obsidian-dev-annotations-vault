Após a leitura desse artigo, recomendo visualizar os artigos das ferramentas de observabilidade:

[[Prometheus]]
[[Open Telemetry]]
[[Prometheus vs Loki vs Tempo vs OpenTelemetry]]
[[OTel, Jaeger, Prometheus e Grafana]]
## O que é Observabilidade?

Na [teoria de controle](https://pt.wikipedia.org/wiki/Teoria_de_controle), a **observabilidade** é definida como uma medida de quão bem os estados internos de um sistema podem ser inferidos a partir do conhecimento das saídas externas
desse sistema. Simplificando, observabilidade é quão bem você pode entender seu sistema complexo, ou seja, em resumo, ao vermos os outputs e entendendo esses outputs, podemos ver o estado do nosso sistema sem estarmos dentro do mesmo. Dessa forma conseguimos, ver dentro de aplicações importantes quais erros estão acontecendo e onde estão acontecendo.

## Observabilidade vs Monitoramento

Irei explicar a diferença entre esses dois tópicos através de alguns bullet points:

- Monitoramento nos mostra que há algo de errado, por exemplo algo da errado e nosso “painel de controle” começa a apitar;
- Monitoramento se baseia em saber com antecedência quais sinais desejamos monitorar, e esses sinais são alertas, métricas;
- Observabilidade nos permite perguntar o porquê de algo está dando errado, por exemplo, a CPU do nosso provedor está alta, a RAM está sendo consumida de forma inapropriada, e assim vemos o porque disso tudo esta trabalhando de forma errônea

## Os 3 pilares da Observabilidade

### 1º Pilar - Métrica

Primeiramente e indo direto ao ponto, métrica é algo claro, é algo extremamente preciso. Métrica é **número**. Por exemplo, a CPU está em 90%, temos **X** de RAM livre, temos **Y** de pods disponíveis, temos **W** de espaço em disco disponível.

Dessa forma, essas métricas conseguem ajudar a guiar a nossa parte de infra, nossa aplicação e também o nosso negócio. E assim, podemos ver dois tipos principais de métricas, as do lado técnico e as do lado de negócios. 

**Métricas de Negócio →** Quantas visitas no site, quantos trials tivemos, quantos cadastros, quantas compras, etc

**Métricas Técnicas →** Quantos status de erros, quantos endpoints temos acima de X tempo, quantas requisições deram exceções

### 2º Pilar - Logs

Os logs são registros imutáveis que possuem data/hora de eventos que ocorrem em um sistema, incluindo erros, alertas e informações de depuração. Possui 3 diferentes formas, podendo ser em formato texto, estruturado ou binário.

Eles fornecem contexto sobre o que está acontecendo no sistema e podem ser utilizados para identificar problemas e rastrear sua origem. Também auxiliam na descoberta de comportamentos emergentes e inesperados que são apresentados por componentes de um sistema distribuído.

### 3º Pilar - Tracing

Os traces (tracing ou rastreamento) são registros detalhados das chamadas entre diferentes componentes de um sistema, incluindo informações como o tempo gasto em cada chamada, entradas e saídas. Eles permitem rastrear e entender a cadeia de eventos que levam a um problema ou lentidão no sistema, identificando a quantidade de trabalho que foi realizada em cada camada do caminho seguido por uma requisição.

Hoje nos cenários de microsserviço, uma requisição pode iniciar em um microsserviço e partir para diversos outros, e do nada pode dar um log de erro, e com o tracing pode saber o passo-a-passo do que aconteceu

![[Observabilidade/Attachments/image.png]]

## Introdução ao Elastic Stack

Primeiramente, se ouvia muito sobre a ELK Stack, com isso vamos desmembrar cada sigla:

- **E** → **Elastic Search**, que é um Search Engine And Analytics, que resumidamente, trabalhar de forma em que iremos jogar alguns dados e ele vai fazer uma busca e analisar de forma rapidíssima
- **L** → **Logstash**, é um processador de dados, através de pipelines que conseguem receber, que veem de múltiplas fontes, transformar e enviar dados simultaneamente para o **Elastic Search**.
- **K** → **Kibana**, é um ferramenta de dashboard, que permite usuários a visualizarem os dados do Elastic Search em diversas perspectivas

### Conhecendo mais adentro da Stack

**Elastic Search**

O Elastioc Search é um search engine e analytics, que foi construído pela Elastic em 2010, que significa que uma vez que temos dados nele, ele consegue fazer uma buscar de forma rápida, ele trabalha com índices, esses dados são documentos JSON e cada item desse dado podemos compor e metrificar.

Ele foi feito em cima do **Apache Lucene**, que é o seu motor.

O Elastic Search é escalável, de forma descentralizada ou distribuída através de shards, ou seja, ele escala horizontalmente, criando várias outras máquinas e ele vai criando índices e duplicando os mesmos índices em todas as máquinas, que possuem redundância de dados, e trabalha com um API Rest.

E como ele trabalha com vários tipos de dados, inclusive geospacial (latitude e longitude), buscas em áreas, trabalhar com mapas.

E também como é uma Search Engine, ele consegue fazer uma application, website e enterprise search.

Podemos usar a parte de logging e analytics de nossas aplicações dentro do Elastic Search também

## Mais sobre o Logstash

De forma beeeem resumida, podemos dizer que o Logstash é uma engine coletora de dados em tempo real. A idéia do Logstash se deu quando os desenvolvedores viram que conseguiam colocar qualquer tipo de log de do Elastic Search, porém colocar o log de em contexto puro não ficaria tão simples em fazer a busca, por exemplo, dentro do log do NGINX temos o IP, a URL de acesso, a data e hora desse acesso, tudo isso em um texto só. Contudo, para fazermos uma busca mais assertiva temos que transformar esse texto em um documento onde temos todos esses campos necessários e todo esse processo é feito por meio de pipelines. E esses dados podem vir várias formas de um TXT, do RabbitMQ, e etc.

Outro ponto, é que o Logstash também trabalha com plugins

## Sobre o Kibana

O Kibana é a ferramenta padrão de visualização e exploração de dados do Elastic Search. Ela é usada para logs, análises de séries, monitoramento de aplicações e inteligência operacional, hoje também temos integração com recursos de AI da Elastic para detecção de anomalias, e também tem a parte do desenvolvedor para testes e buscas. E com todo aquele processo de documentos do Elastic Search, podemos criar regras para fazermos agregações e filtragem desses dados dentro do Kibana.

Dentro dessa ferramenta, podemos gerar dashboard, criar gráficos interativos para monitorarmos, ver métricas de negócio, etc.

## Qual a diferença entre o ELK Stack e Elastic Stack?

O grande ponto de diferença é o **Beats**, que é um projeto da Elastic em 2015, que a idéia principal dele é ser um “Lightweight Data Shipper” (traduzido diretamente é um entregador de dados leve). Basicamente, foi percebido que cada dia mais se tinha mais locais para ingerir dados e todos os tipos, e por padrão trabalhava-se somente com o Logstash, o Logstash é um pouco “tricky”, e dessa forma foi lançado Beats.

![[Observabilidade/Attachments/image 1.png]]

Ele trabalha como um agente coletor de dados, ou seja, ele pega o dado de algum lugar e pode jogar para o Logstash ou para o Elastic Search.

<aside>
⚠️

**DISCLAIMER!** O Logstash está caindo em desuso, inclusive a Elastic tem serviços em cloud dentro do provider que estamos usando e a stack que é provisionada vem sem o Logstash.

</aside>

O grande pulo do gato do Beats é que ele é diversificado, então temos os seguintes tipos de Beats:

- Logs;
- Métricas;
- Network Data;
- Audit Data;
- Uptime Monitoring;

E hoje, mediante a feitura desse documento, podemos ja ter o Beat unificado.

Outra sacada, é que podemos criar um Beat específico para o que precisamos 😲.

Com essa, adição do Beats, temos o Elastic Stack!

![[Observabilidade/Attachments/image 2.png]]