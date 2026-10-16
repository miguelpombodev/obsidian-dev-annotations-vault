Cache  é como um atalho para caminhos frequentemente viajados, ele economiza tempo e recursos.
E em ambientes de APIs, caching pode melhorar muito a performance e escalabilidade, reduzindo a carga em bases de dados e servidores.

## Por que há diferentes estratégias de caching?
Existem diferentes estratégias de caching porque apenas algumas abordagens conseguem atender aos diversos requisitos de todas as aplicações.
A escolha da estratégia depende de fatores como os padrões de acesso aos dados, requerimentos de consistência, tolerância a latencia, e a natureza das operações de escrita e leitura.

## Strong consistency VS Performance
Algumas aplicações, como sistemas financeiros, priorizam consistência acima de performance, tornando estrategias como **write-through** mas adequadas.
Por outro lado, aplicações que requerem alta performance porém que toleram pequenas inconsistências podem se beneficiar de **caching write-back**.

Write-through é mais usado em cenários como atualizações de stocks em tempo real em e-commerces, onde os dados são escritos no cache e no banco de dados para assegurar a forte consistencia apesar do custo alto.

## Read-Heavy VS Write-Heavy Workloads
Sistemas com predominante operações de leitura pode utilizar estratégias de **read-through** or **cache-aside**. Sistemas com mais operações de leitura pode utilizar estratégias como **write-around** ou **write-back** para reduzir pressão sobre cache. Website de notícias populares geralmente são cacheadas para reduzir latencia e otimizar perfomance.

## Data freshness
Certas aplicações, como plataformas de stock trading, precisam de dados em tempo real, favorecendo estrategias que frequentemente atualizam o cache. Em contrapartida, dados estáticos ou que raramente mudam, como arquivos CSS, imagens, ou conteudo em CDN, performam melhor com estratégias de caching que reduzem a frequência de atualização.

![[Pasted image 20260819200032.png]]

## A perfomance realmente muda entre estas estratégias?
Sim, a performance varia significadamente entre as estratégias, e a escolha pode impactar a latencia, throughput e a utilização de rescursos.

### Estratégias de leitura
#### Cache-Aside ou Lazy-Loading
Oferece flexibilidade e controle sobre o cache mas requer que a aplicação lide com a logica de caching. Perfomance também depende em o quão bem as decisões de cache se alinham com os padrões de uso.

#### Read-through
Otimizado para aplicações com leituras frequentes e padrões de acessos previsíveis. Os acessos ao cache providenciam leituras de baixa-latencia, mas as falhas de cache pode aumentar a latencia devido as operações de busca e atualização.

![[Pasted image 20260819200838.png]]

> Performance aqui é um trade-off entre velocidade, consistência de dados e uso de recursos. Testes e monitoramento são essenciais para determinar a  melhor estratégia para o workload que tem

### Estratégias de escrita
#### Write-through
A latência de leitura é maior devido as operações síncronas tanto no cache quanto no data store. Porém, a latencia de escrita permanece baixa desde que o cache se matenha atualizado.

#### Write-around
Reduz a latencia de escrita para gravações diretas no data store, porém pode causar falhas de cache, e assim aumentando a latencia de leitura

#### Write-back ou Write-behind
Escritas rápidas devido a atualizações rápidas, porém há riscos de perda de dados e requer mecanismos sofisticados para gerenciar falhas.

![[Pasted image 20260819202326.png]]

