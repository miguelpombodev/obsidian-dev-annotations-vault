*Fontes: https://opentelemetry.io/docs/*

# Telemetria

Primeiramente, antes de falarmos sobre a ferramenta OpenTelemetry, devemos colocar o conceito de telemetria.

> **Dados de telemetria referem-se à coleta e transmissão automáticas de logs, métricas, eventos e traces de fontes remotas utilizando sensores e protocolos. Esses dados são enviados para sistemas centralizados para monitoramento e análise, permitindo um gerenciamento e controle eficaz do sistema.**
> 

*Traduzido de: https://www.logicmonitor.com/blog/what-is-telemetry*

## Span

Spans são espaços de tempo onde vemos o início e fim de certos blocos de códigos, métodos, chamadas HTTP, leitura ou escrita em bancos de dados/discos.

![[OTEL_image.png]]

Exemplo de um span

## Customização das Informações

Além das métricas que já temos por padrão, por exemplo, sobre uso de CPU, uso de RAM, espaço em disco, latência de rede, uma das estratégias que nos ajudam na relação produto-negócio, ao usar a telemetria é a customização de informações. Podemos criar métricas que nos ajudem a identificar certos padrões em uma aplicação e criar estratégias com que possamos melhorar o negócio do produto, por exemplo, quantos usuários saem antes de finalizar uma compra, registros de chat de ajuda, etc. Mas também podemos ter métricas técnicas customizadas, como quantidade de requisições por minuto.

## Lock In

Um assunto interessante é que quando usamos ferramentas de tracing como **ElasticAPM**, **DataDog**, **NewRelic**, etc, podemos ter um ponto bem complicado chamado **lock-in**, nesse caso teríamos um vendor lock-in que seria a dependência específica de um serviço, de uma ferramenta, ou produto, e isso seria uma grande problema caso precisássemos fazer a troca para outro vendor.

Sabendo disso o OpenTelemetry vem para nos ajudar a padronizar as informações das métricas que teríamos desses tracers.

# O que é o OpenTelemetry?

O OpenTelemetry é um framework de observabilidade para softwares cloud native, criado pela CNCF (**Cloud Native Computing Foundation**), ou seja, o OpenTelemetry cuida de tudo o que precisamos para trabalhar com observabilidade, que são logs, métricas e tracing, além de trabalhar com **instrumentação, geração, coleta e exportação de dados de telemetria.**

O OpenTelemetry disponibiliza diversos pacotes e libs para trabalharmos com o seu padrão, ou seja, suas APIs e seus SDKs.

# Componentes principais do OpenTelemetry

Nessa seção, iremos passar pelos principais componentes que estão dentro do OpenTelemetry, para nos dar um embasamento maior sobre a ferramenta.

## Especificações

Antes de qualquer tipo de implementação dentro do OpenTelemetry, precisamos ver as especificações da aplicação que trabalhamos, para que não tenhamos dados totalmente misturados e sem sequer um tipo de padrão. O segundo ponto é sobre as SDKs, ou seja, temos SDKs para todos os tipos de LPs e frameworks, pois também cada LP tem um tipo de tracing específico.

## Collector

O Collector é um agente ou serviço, mas como assim? O Collector é a peça fundamental para resgatar os dados de telemetria gerados pela aplicação para processar e/ou enviar para outros vendors, porém ele pode ser um agente que vai ficar rodando dentro da aplicação, ou um sidecar que fica em paralelo à aplicação, ou até mesmo um serviço a parte somente para coletar esses dados.

O Collector também é uma pipeline, onde ele pode pegar um certo dado e fazer diversas coisas  em diversas etapas, até que os dados estejam prontos para o vendor, por exemplo, vamos pensar em um dado sensível de um usuário cai em dos logs da aplicação, pode fazer com que caso isso aconteça, podemos fazer algum tratativa à esse dado.

Essa pipeline pode ser nos formatos:

- Processor;
- Receiver;
- Exporters

Além disso, tudo o Collector é vendor-agnostic, ou seja, tanto faz se temos um DataDog, ElasticAPM, Jaeger, Grafana Tempo, XPTO, etc… Como podemos ver, não teremos lock-in sob essa ferramenta.

![[OTEL_image 1.png]]

## Sem Collector

Vamos usar o exemplo abaixo, onde dentro de um servidor temos uma app e dentro dela ja temos as libs, ou seja as SDKs do OTEL instaladas. Junto disso, teremos dois vendors, suponhamos que o vendor 1 é um ElasticAPM, onde recebemos as metricas e os tracings, e o vendor 2 é um DataDog, que recebe os logs.

![[OTEL_image 2.png]].png] ]

Esse tipo de configuração é totalmente viável, desde que façamos a configuração adequada. O ponto importante é que a aplicação além de fazer tudo o que precisa, ela precisa fazer as requests externas para um vendor e esse vendor pode estar lento ou com uma indisponibilidade.

## Collector no modo agente

Repetiremos a mesma estrutura base, de servidor-aplicação, porém teremos paralelamente um aplicação que auxilia a aplicação principal, o que podemos chamar de “sidecar”, que estará na mesma rede, sendo assim terá menos latência.

![[OTEL_image 3.png]]

Nesse caso, ao invés de fazermos a conexão direta com os vendors, mandamos a informação para o collector, e ele recebe, processa e exporta, e assim podemos também escolher qual o vendor vamos usar. E podemos fazer isso, com não somente 1 aplicação mas com N apps que possamos ter.

![[OTEL_image 4.png]]

## Collector no modo server

Retornemos para a estrutura base, porém agora entre o servidor e os vendors, temos o Collector Server, onde precisamos subir uma máquina ou container a parte, e assim ele vai trabalhar como se fosse um middleware de serviço

## Libs

As libs do OTEL são vendor-agnostics também, assim como o Collector. Dentre as libs que temos no OpenTelemetry, as que mais podemos falar são as de Tracing, Logs e Auto Tracing (Esse comportamento cai no assunto sobre **AutoInstrumentação**).

<aside>
⚠️

**DISCLAIMER!**

Devemos lembrar que o OpenTelemetry é uma ferramenta que está continuamente sendo desenvolvido, então podemos ter em algumas LPs, algumas dessas funcionalidades podem não ter sido desenvolvidas, ou estão em alpha version.

</aside>

# Instrumentação

Instrumentação é maneira que temos de conseguir lidar com o que temos na aplicação que apartir de um comportamento conseguimos gerar dados para enviar ou não para um Collector.

Um ponto de conhecimento que devemos ter que instrumentar uma aplicação pode ser complexo, pois dentro da aplicação temos consumo de memória, de I/O, tempo de startup, cold start, chamadas à DB, enfim… E fazer isso tudo manualmente é impossível.

O grande ponto é que trabalhando com o OTEL temos uma instrumentação automática, ou seja, teremos um lib que ficará “sugando” tudo que aparece da aplicação e cuspindo para o collector, e essas informações obtidas podem ou não serem customizadas (**depende do status da implementação**).

<aside>
⚠️

**Essa instrumentação automática não está disponível para todas as LPs!**

</aside>

O OpenTelemetry é um framework de observabilidade para softwares cloud native, criado pela CNCF (**Cloud Native Computing Foundation**), ou seja, o OpenTelemetry cuida de tudo o que precisamos para trabalhar com observabilidade, que são logs, métricas e tracing, além de trabalhar com **instrumentação, geração, coleta e exportação de dados de telemetria.**