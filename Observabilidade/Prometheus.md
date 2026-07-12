# Conceitos básicos

O Prometheus trabalha com métricas e alertas para monitorar nossas aplicações, que através disso tudo conseguimos ter alguns insights.

Ou seja, o Prometheus é um toolkit de métricas, monitoramento e alerta open-source que foi criado pela SoundCloud, porém agora ele roda de forma independente e faz parte da CNCF (**Cloud Native Computing Foundation**), sendo assim ele trabalha sem lock-in, independente de qual for a cloud que usemos e também conseguimos trabalhar com ele de forma on-premise.

## Formatos de dados

Uma coisa interessante é que o Prometheus consegue capturar dados dimensionais, ou seja ele além de capturar dados “crus” das métricas, também conseguimos diferentes formatos de dados, além de  obter dados agrupados e no formato de “histograms”

![[Prom_image.png]]

exemplo de um histograma

## Consultas poderosas

Com o Prometheus conseguimos também criar consultas bastante poderosas, diversas e performáticas, com uma linguagem própria

## Grafana

Por conta do Prometheus ser um excelente coletor de dados e métricas, porém não conseguimos visualizar esses dados coletados de forma nativa, ou seja, nele mesmo, para isso precisamos usar algum sistema que gerencie dashboards. Geralmente a ferramenta usada para a visualização dos dados do Prometheus é o **Grafana**.

## Storage

O Prometheus também tem um gerenciamento de armazenamento muito interessante, de forma que ele consegue garantir precisão nas informações mais recentes, perde nas mais antigas porém ao mesmo tempo ele comprime as informações mais antigas para reduzir o espaço em disco

## Simples

O Prometheus é de simples implementação e bem “*straightforward*”, mas não significa que ele é de facil usabilidade e extração de dados.

## Alertas Inteligentes

Com um sistema de alertas bem inteligente, conseguimos encadear alertas, por exemplo, em um sistema de vendas pensando em temos um encadeamento de alertas de “1 hour server down” e outro alerta de “sem vendas por 1h”, o Prometheus invés de triggar dois alertas, ele consegue otimizar e ser inteligenter o bastante para triggar somente 1 alerta nesse caso.

## Diversidade

Outro ponto legal, é que temos diversão soluções para integrar o Prometheus com as maiores e principais soluções do mercado, como SGBDs, OSs, Web Services, Proxies, Kubernetes, etc. E fazemos isso através de clients ou libs que conectam esses demais serviços.

# How it works?🤔

Vamos imaginar que temos uma aplicação, e nessa aplicação e também temos um serviço monitor, por exemplo um APM, e dentro da nossa aplicação temos um agente, que nós mesmo instalamos, e que de tempos em tempos vai enviando as informações para o serviço de forma ativa.

![[Prom_image 1.png]]

Porém também temos uma outra forma de enviar esses dados para o nosso serviço, onde que ao inveś de instalarmos um agente dentro da nossa aplicação, fazemos com que o próprio serviço, também em um dado range de tempo, fique dando um “*pull*” dentro da nossa aplicação e e registra a informação dentro do serviço. Dessa jeito, temos uma forma passiva, onde nossa aplicação precisa ficar mandando a cada evento e sim o proprio serviço “*attached*” que faz isso.

![[Prom_image 2.png]]

<aside>
☝🏽

O importante é que a aplicação se mantenha viva e que consiga disponibilizar um endpoint onde o serviço consiga receber as métricas necessárias

</aside>

# Prometheus VS *pull*

Na seção anterior, vimos como o Prometheus trabalha, porém agora iremos falar um pouco mais sobre ele.

O *pull* que o Prometheus faz dentro da aplicação é feito **via HTTP**, mesmo sendo estranho pois como dito novamente, conseguimos pegar métricas de banco de dados, OSs, e tudo mais.

Como podemos ver abaixo, a “configuração” ilustrada esta dizendo que cada 15s o Prometheus irá chamar o enpoint “**/metrics**¨ (esse nome pode ser qualquer outro, mas é meio que convenção).

![[Prom_image 3.png]]

Porém precisamos adaptar esse endpoint para um formato onde o Prometheus consiga entender e parsear os dados (infelizmente não é **TOTAL** de qualquer jeito). Veja o exemplo:

![[Prom_image 4.png]]

Essa forma pode assustar em uma primeira vista mas conseguimos gerar essas informações de forma automática através de SDKs que estão disponibilizadas para qualquer LP.

# Exporters

Quando estamos trabalhando com a nossa aplicação, precisamos de métricas e essas métricas normalmente são relevantes ao negócio, como por exemplo:

- Quantidade de compras;
- Tempo de resposta no processo de compra;
- Quantidade de usuários logados;
- Utilização de feature X;
- Busca específica no site

Essas informações ajudam ao business a tomar algumas decisões sobre o produto, campanhas e etc.

O problema é que nessa situação conseguimos implementar alguma funcionalidade para obter essas informações, e quando essas informações precisam vir de algum serviço que não foi a gente que desenvolveu? Exemplo:

- MySQL;
- NGINX;
- Linux Server;
- Kubernetes;
- etc…

Com isso o Prometheus desenvolveu algo chamado “**Exporters**”.

Vamos imaginar que precisamos das métricas de um servidor Linux, dessa forma iremos colocar um exporter no meu desse caminho e ele funcionar da seguinte maneira, o exporter vai disponibilizar um “servidor web” com o endpoint “/metrics” e quando precisarmos pegar as informações do servidor Linux, o Prometheus irá fazer o mesmo pull que faria em uma aplicação comum, só que dessa vez no exporter que configuramos:

![[Prom_image 5.png]]

Assim será para qualquer outro tipo de serviço que seja fora da nossa aplicação que estamos trabalhando.

E se caso, não tenhamos disponibilizado um exporter para o serviço que necessitamos, esse mesmo exporter pode ser criado.

# Arquitetura do Prometheus

Algo que o Prometheus tem dentro de sua arquitetura é algo que ele chama de “Prometheus Server”, onde esse server é separado em 3 partes:

- Retrieval → Esse componente que vai receber as informações coletadas e jogar dentro do TSDB;
- TSDB → Time Series Database é o componente que faz o processo de armazenamento das informações no formato de Time Series, e esse banco de dados é usado pois por conta de trabalharmos com métricas, essas métricas vem com o valor + o timestamp dela;
- HTTP Server → Esse servidor HTTP fica ligado onde até mesmo **PROMETHEUS MESMO** consegue se monitorar

![[Prom_730d49ed-5364-4256-8f24-f6f3b208b145.png]]

Além desses componentes iniciais dos quais estamos falando, é claro que temos outros porém precisamos fazer um levantamento antes de tudo, quando falamos que o Prometheus monitora nossas aplicações, é óbvio que temos que “dizer” quais aplicações ele tem que monitorar, certo?

Bom, sendo assim temos que passar o nome da aplicação, a URL, o tempo de monitoramento e até tudo bem, e quando esses serviços são dinâmicos ou quando temos um autoscaling? Como fazemos isso? É por conta disso que o Prometheus também consegue se conectar e falar com os services discovers para que ele consiga monitorar todas as máquinas que estão ligadas naquele service discorver para ele conseguir ter o retrieve das informações.

Como podemos ver abaixo, temos na imagem um Kubernetes e o Kubernetes tem o seu próprio service discover, assim o Prometheus vai monitorar os pods, vai se conectar ao service discover e vai saber todos os pods que estão de pé e começar a ter as informações.

![[Prom_image 6.png]]

Outro ponto de grande importância, como vimos anteriormente, para que o Prometheus comece a ter acesso as informações ele tem o componente de Retrieval e esse componente tem a parte de “pulling metrics” e guardando dentro do TSDB, mas além disso o Retrieval tem seus **jobs** que vão ser executados no targets(aplicações) ou nos exporters.

Agora vamos pensar seguinte cenário, temos um job que roda todo dia de madrugada durante 5min e precisamos dessas métricas também, como conseguimos pegar as informações de um serviço que não esta no ar o tempo todo ou pouco acessado? Respondendo esse dúvida, o Prometheus tem um outro mecanismo chamado de “**Pushgateway**”. Pegando o exemplo acima novamente, quando esse job é realizado na hora que ele acontecer, ele manda suas informações para esse Pushgateway e depois ele pode ser desligado sem problema algum, e o Prometheus vai lá e pega essas infos, como se fosse um reservatório para esse job.

![[Prom_image 7.png]]

Iremos agora partir para os alertas no Prometheus, e esse sistema de alerta trabalha da seguinte forma, o sistema de alerta se conecta ao Prometheus server e as informações irão para um “Alert Manager” que vai fazer as notificações para as plataformas necessárias.

E por outro lado o HTTP Server do Prometheus vai servir da forma que consigamos extrair as informações dele e utilizá-las, por exemplo dentro do Grafana.

Agora temos todo o esquema da arquitetura do Prometheus:

![[Prom_image 8.png]]

# Trabalhando com dados

Antes de falarmos como os dados são trabalhados, quais dimensões são usadas e tudo mais, vamos falar sobre armazenamento dentro do Prometheus

Como vimos na seção anterior, o Prometheus trabalha com um TSDB (Time Series Database) ou seja, ele guarda as informações contiguamente conforme o tempo passa. Outro ponto importante, é que conforme o tempo passa e os dados mudam, o Prometheus consegue fazer um armazenamento um pouco mais eficiente, e em relação à isso mesmo que o serviço do Prometheus caia e depois restarte, ele consegue retomar de onde ele parou para fazer esses ajustes.

<aside>
☝🏽

Porém sempre lembrando, quanto mais nova a informação mais teremos precisão, quanto mais antiga, menor será a precisão.

</aside>

## Labels

Um trabalho muito comum do Prometheus é o com “labels” para setar propriedades em métricas, traduzindo, conseguimos colocar algo como se fosse “submétricas” dentro de uma métrica principal. Exemplo, temos uma métrica de diversos erros (**400,401,402,403, 404, 500**), mas quando esse erro acontecer queremos guardar o tipo do erro ocorrido, então teremos uma label de tipo de erro(erro_type) e essa informação será guardada. Isso ajuda para evitar de criarmos uma métrica para cada tipo de erro.

Vamos ver um exemplo de como é visto um valor pode ser armazenado no Prometheus:

![[Prom_image 9.png]]

Podemos achar que ao somarmos todos os valores da coluna “Erro 500” teremos o valor total de eventos ocorridos, porém não é bem assim que o Prometheus funciona, o jeito que ele faz é que durante o tempo do primeiro timestamp e o segundo timestamp tivemos apenas 3 erros, e não que tivemos 12 erros em um e depois tivemos 15. Ele funciona como um acumulador.

# Tipos de Métricas

Aqui veremos quais são os principais tipos de métricas que o Prometheus consegue armazenar e que é afirmado que se tivermos esses principais tipos de métricas (4 ao todo), conseguimos obter as informações que precisarmos.

## Counter

Esse tipo é incremental, nunca iremos ver um valor desse tipo decrementando em algum momento, um exemplo disso é a imagem de registros na seção passada. E caso o Prometheus venha a ter um “down state”, ele vai ver o último registro e voltar a incrementar baseado nesse último valor

Exemplos:

- Quantidade de visitas em um site;
- Quantidade de vendas;
- Quantidade de erros

## Gauge

Esse outro tipo pode possuir variações de valores conforme o tempo vai passando, ou seja, um dado que pode ficar aumentando, diminuindo ou até mesmo estabalizado.

Exemplos:

- Quantidade de memória usada;
- Quantidade de usuários online;
- Quantidade de máquinas ativas

## Histogram

Esse tipo trabalhamos com ele quando precisamos distribuir dados com uma certa frequência, ou seja, ela pega amostras e baseando-se nessas amostras, o Prometheus faz seus calculos para que consiga plotar essa informação. E além de termos essas informações baseadas em frequência também conseguimos agregar elas.

![[Prom_image 10.png]]

## Summary

Esse tipo é bastante similar ao Histogram, porém aqui os valores são calculados no servidor de aplicação e não no Prometheus, como é feito nas demais, por conta disso é mais comum ver o Histogram sendo usado.

# PromQL

O PromQL (Prometheus Query Language) é basicamente o SQL do Prometheus, então assim que precisarmos consultar as informações já registradas dentro do Prometheus conseguimos usar essa linguagem que é bastante explícita.

Exemplo:

- **http_requests_total** → Aqui conseguimos ver a quantidade total de requests HTTP;
- **rate(http_requests_total[5m])** → Aqui temos a quantidade total de requests HTTP porém num espaço de tempo de 5 minutos;
- **http_requests_total{status!~”4..”} →** A quantidade total de requests HTTP que sejam diferentes de 400 ****