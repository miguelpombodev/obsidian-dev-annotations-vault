# Large Scale e Complex Archs

## Large Scale Architecture

Arquiteturas de Larga Escala ou Large Scale Architecture são arquiteturas que já foram projetadas para suportar sistemas que operam em um nível grande de volumes de dados, números de usuários, ou ambos. Os algoritmos desses sistemas podem ate mesmo serem complexos porém a nível arquitetural esses sistemas são simples.

São caracterizadas pela necessidade pela alta disponibilidade, perfomance e eficacia durante picos e geralmente utilizam conscientemente ou inconscientemente patterns de arquitetura e boas práticas existentes.

E geralmente envolvem diferentes tecnologias de compute, storage e engine de base de dados (**in-memory** e **block device**). Por exemplo, podemos ter mais de um tipo de banco de dados, um Redis, CDN, então um ponto que temos nas Large Scale Architectures é o “dividir para conquistar”, ou seja, dividimos em diversos pedaços a arquitetura para “conquistar” o atendimento àquele diverso número de usuários que podemos ter.

E além do software, também temos um “Control Plane”, ou seja, ele atua como o "cérebro" de um sistema distribuído, separando os processos de tomada de decisão (**gerenciamento**, **configuração**, **orquestração**) do "data plane", que lida com o tráfego de usuários e a execução da workload.

<aside>
💡

 palavra de define as LGAs é **OVER ENGINEERING**, pois acabamos pensando tanto em colocar um ferramental e tecnologias, e as vezes nem precisávamos de todo aquele poder.

</aside>

![[Software Architecture/Attachments/image.png]]

Exemplo de LGA

## Complex Architectures

Arquiteturas Complexas ou Complex Architectures são arquiteturas que tem diversas camadas, componentes interdependentes, uma variedade de tecnologias (polyglot), também podemos ter uma quantidade de comunicação e diversos tipos de comunicações (SOAP, RESTFul, gRPC, async, mensagerias, batches).

Dentro desses sistemas, as regras de negócios são bem complexas, ou seja, estamos sempre convivendo com o passado, não conseguimos reescrever um sistema e parar de entregar features. Basicamente, quando podemos refatorar esses sistemas, precisamos atuar no clássico “construir o avião em movimento”.

Esse tipo de arquitetura é muitas vezes encontradas em entreprises, empresas de 20, 30, 50 anos.

<aside>
💡

A palavra de define as CAs é **OVER THINKING**, pois acabamos pensando demais na complexidade do sistema que acabamos não abstraindo certos passos e simplificando o sistema em si.

</aside>

![[Software Architecture/Attachments/image 1.png]]

Exemplo de Complex Architecture

---

Independente da arquitetura, seja ela uma LGA ou CA, elas podem ser complexas ou não, o ponto de ajuda que temos é que os **patterns de arquitetura** irão nos ajudar em ambos os casos.

# Principios de Sucesso de Arquitetura de Software

![[Software Architecture/Attachments/image 2.png]]

Classificar se nossa arquitetura tem sucesso ou não é muito relativo, porém quando atendemos os pontos da imagem acima, conseguimos dimensionar o sucesso da arquitetura, de modo geral, falando em conceito de tecnologia.

## Planejamento de Capacidade

Aqui falamos da “capacity” do nosso ambiente, estamos falando da capacidade de aumento de número de recursos, de fazer autoscale ou não, inclusive de atender os usuários em um hardware falho. Não podemos depender do que temos de disponível, mas sim, pensando sempre em ter mais.

## Observabilidade

Esse ponto é um dos mais importante, pensando em “reliability” (confiança) do sistema, isso desde o nível de aplicação até o nível de infraestrutura. Precisamos ter noção da jornada de um fluxo como um todo, qual são os traces, spans das chamadas. E além disso tudo, precisamos ter alertas e alarmes para quando tivermos um erro ou algo inesperado. Em resumo, precisamos ter uma visão completa das situações de erros e “warnings”.

O planejamento de capacidade depende muito nos dados da observabilidade das aplicações e arquiteturas, para entendermos quais os pontos de melhora.

## Otimização de custos

A otimização de custos está totalmente atrelada aos dois pontos anteriores (“planejamento de capacidade” e “observabilidade”), e as vezes essa melhora otimização se é feita em pontos não palpáveis, como por exemplo, atualização de um tecnologia 

## Operações

Esse tópico é como vamos ver a operação de todos os tópicos aqui mostrados, ou seja, no dia 2. 

## Release Engineering

É a capacidade de fazermos um deployment da aplicação, de forma que minimize o impacto financeiro (o quanto a aplicação custa) e melhorar o jeito que entregamos features e disponibilizamos a nova versão da mesma.

## Segurança

Ponto mais importante, hoje em dia, em aplicações de alto nível, que é onde asseguramos que a aplicação esta segura sobre qualquer tipo de ataque externo, como DDoS, XSS, onde temos criptografias, protocolos seguros

## Confiabilidade

É capacidade de estar acima do SLA de disponibilidade, de resiliência, ter uma estratégia de backup/restore/rollback, estratégia de disaster recovery. Esse é ponto é sobre o ato da aplicação ser confiável. Segue os pontos de atenção:

- **Consistência** → Entregamos os mesmos resultados ao realizar uma operação ou tarefa específica, o quão consistentes estão os nossos dados;
- **Durabilidade** → Sabermos que após que persistirmos um dado, não iremos perde-lo, ou seja, resistência a períodos prolongados de uso ou operações sem degradações ou falhas;
- **Tolerância a falhas** → Resistência a falhas e erros e a capacidade de se recuperar de maneira graciosa ;
- **Previsibilidade** → Os usuários podem prever como o sistema de comporta, seja conforme esperado ou não. (aqui o tópico de “Observabilidade” é essencial);
- **Disponibilidade** → Disponível e acessível aos usuários sempre que necessário, sem tempo de inatividade ou interrupções inesperadas. E aqui não é somente sobre a aplicação estar somente “up” mas como seus recursos também

# Planejamento alinhado ao negócio