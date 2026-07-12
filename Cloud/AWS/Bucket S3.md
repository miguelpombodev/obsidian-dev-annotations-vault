O **Amazon Simple Storage Service** foi lançado no dia 14 de março de 2006, como primeiro produto lançado pela AWS, um dos mais antigos do Cloud Provider.

O Amazon S3 é um serviço global de armazenamento de objetos (arquivos) que oferece escalabilidade, disponibilidade de dados, segurança e desempenho, além de fornecer recursos de gerenciamento de otimização, organização e configuração de acessos aos dados para atender aos requisitos especificados pelo negócio.

O S3 oferece uma ampla variedade de classes de armazenamento para diferentes casos de uso, e também possui recursos de ciclo de vida de objetos(otimização de custos) e fornece recursos para auditoria e gerenciamento de acesso a seus buckets e objetos.

O S3 fornece um armazenamento mais durável na nuvem. Com base em sua arquitetura exclusiva, o S3 foi projetado para exceder uma durabilidade de dados de 99,99999999999%. Além disso, o S3 armazena dados de forma redundante em um mínimo de 3 zonas de disponibilidade por padrão, fornecendo resiliência integrada contra desastres generalizados.

Os clientes podem armazenar dados em um única AZ para minimar o custo ou a latência de armazenamento, em várias AZs para maior resiliência contra perda permanente de um data center inteiro ou em várias regiões da AWS para atender aos requisitos de resiliência geográfica.

Se caso não tivermos requisitos de residência de dados que não podem ser atendidos por uma região da AWS existente, poderá usar a classe de armazenamento S3 Outposts para armazenar seus dados dos S3 on-premises.

# Buckets

Buckets são containers para o armazenamento de objetos.

Por padrão, podemos armazenar um número ilimitado de objetos em um Bucket, porém podemos ter apenas 100 Buckets por cada conta, esse número é limitado por conta do serviço de quota, entretanto podemos pedir expandir esse número abrindo uma solicitação para a AWS, caso necessário.Cada bucket deve ter um nome único globalmente, pois cada nome de bucket é um nome de DNS.

Os nome dos buckets identificam a conta responsável por cobranças de transferências de dados e armazenamento.

<aside>
🚨

No S3, o mais caro não são os armazenamentos e sim as transferencias de dados.

</aside>

## Tipos de Buckets

- Buckets de uso geral;
- Buckets de diretórios (recomendados para aplicações sensíveis ao desempenho (latencia de PUT e GET));

<aside>
⚠️

Os bucket de diretórios que não tiverem nehuma atividade de solicitação por um período de 90 dias, passarão para um estado inativo. Durante esse período de transição, as leituras e gravações retornarão um código de erro **HTP 503 SERVICE UNAVAILABLE**

</aside>

![[S3_image.png]]

## Chaves

Os objetos são as entidades fundamentais armazenadas no Amazon S3. Um objeto é identificado exclusivamente em um bucket por uma chave (nome) e um ID da versão (se o versionamento do S3 estiver habilitado).

Uma chave de objeto (ou nome da chave) é um identificador exclusivo de um objeto em um bucket. Cada objeto em um bucket tem exatamente uma chave. A combinação de um bucket, chave de objeto e, opcionalmente, o ID da versão identifica exclusivamente cada objeto.

Exemplo de chave:

![[S3_image 1.png]]

**photos/ozzy.jpg** → objeto;

**black-sabbath** → nome do bucket;

**us-east-1** → região;

![[S3_image 2.png]]

# Classes de Storage

Existem 9 classes de storage:

![[S3_image 3.png]]

## S3 Standard

Esta classe é a classe padrão de armazenamento para uso geral, assim que fizermos um upload para um bucket no S3, se não tivermos mudado as configurações, o arquivo vai para o armazenamento padrão.

O S3 Standard oferece um armazenamento de objetos com altos níveis de resiliência, disponibilidade e performance para dados acessados com frequência.

Como fornece baixa latência e alto throughput, o S3 Standard é adequado para uma grande variedade de casos de uso, como aplicações na nuvem, sites dinâmicos, distribuição de conteúdo, aplicações móveis e de jogos e análises de big data.

<aside>
☝🏽

**Throughput** refere-se ao volume médio de dados que podem realmente 
passar pela rede em um período específico. Ela indica o número de 
pacotes de dados que chegam ao destino com êxito e a perda de pacotes de
 dados.

*fonte: https://aws.amazon.com/pt/compare/the-difference-between-throughput-and-latency/*

</aside>

Projeto para oferecer uma disponibilidade de 99,99% com um SLA de disponibilidade de 99,99%.

## S3 Intelligent-Tiering

Esta classe foi feita para acesso desconhecido ou dinâmico, ou seja, se temos uma aplicação ou se temos uma quantidade de dados que são armazenados no bucket mas não sabemos como são acessados ou se são acessados de forma totalmente dinâmica essa camada será a melhor estratégia.

### **Porém, por que?**

Por que o S3 Intelligent-Tiering é o primeiro armazenamento em nuvem que reduz automaticamente os custos de armazenamento em um nível de objeto granular, movendo automaticamente os dados para o nível de acesso mais econômico com base na frequencia de acesso, sem impacto ao desempenho, taxas de recuperação ou sobrecarga operacional.

O S3 Intelligent-Tiering armazena objetos automaticamente em 3 níveis de acesso:

- Acesso frequent (default);
- Acesso infrequente (acesso até 30 dias);
- Acesso instantaneo ao arquivamento (acesso até 90 dias)

Projeto para oferecer uma disponibilidade de 99,99% com um SLA de disponibilidade de 99,99%

Além de ter uma pequena cobrança mensal de monitoramento e automação.

### Como funciona?

![[S3_image 4.png]]

## S3 Express One Zone

Esta classe foi feita para alto desempenho, esmiuçando o conceito, é uma classe de armazenamento de alto desempenho com zona de disponibilidade única criada especificamente para oferecer acesso consistente a dados na velocidade de um dígito em milissegundos para seus dados acessados com mais frequencia e aplicações sensíveis a latencia.

A S3 Express One Zone pode melhoras as velocidades de acesso ao dados em 10 vezes e reduzir os custos de solicitação em 50% em comparação com a S3 Standard, isso tudo pois ele esta vinculado em apenas uma zona de disponibilidade e não mais para 3 zonas.

Podemos selecionar qualquer data center dentro da região para armazenarmos nossos dados.

Projeto para oferecer uma disponibilidade de 99,95% com um SLA de disponibilidade de 99,99%.

### Como funciona?

![[S3_image 5.png]]

## S3 Standard - Infrequent Access

Esta classe é para somente acesso infrequente, indicada para dados acessador com menos frequência, mas que exeigem acesso rápido quando necessários.

Um ponto muito importante é que a categoria S3 Standard-IA oferece os altos níveis de resiliência e throughput e a baixa latência da categoria S3 Standard, com taxas reduzidas por GB de armazenamento e GB de recuperação.

Esta classe também armazena os dados em pelo menos 3 AZs (Zonas de Disponibilidade)

Projeto para oferecer uma disponibilidade de 99,99% com um SLA de disponibilidade de 99,99%.

## S3 One Zone - Infrequent Access

Esta classe trabalha da mesma forma que a S3 Standard-IA porém em somente uma AZ, a diferença é esta classe tem um custo 20% S3 Standard-IA.

Ideal para clientes que querem uma opção de menor custo para dados acessados com pouca frequencia, mas não precisam da disponibilidade e da resiliência S3 Standard ou S3 Standard-IA.

## S3 Glacier Instant Retrieval

Primeiramente, precisamos saber o que é Glacier, ou seja, armazenamento frio ou mais conhecido como arquivo morto, com isso temos vários tipos de Glaciers na AWS
No caso que veremos, o Glacier Instant Retrieval é para acesso instantaneo, ou seja, é uma classe de armazenamento onde podemos recuperar nossos dados de forma instantanea. 

E dessa forma, conseguimos ter uma economia de até 68% nos custos de armazenamento em comparação com o uso da classe S3 Standard-IA, porém os dados dentro do Glacier só poderão ser acessados uma vez por trimestre.

<aside>
☝🏽

**DICA**: o resumo do Glacier, independente dos niveis, é armazenar meus arquivos por um longo período e não vou precisar acessá-los com frequencia e quero economizar.

**Exemplos**: gravações de call center, arquivos de auditoria

</aside>

### Como funciona?

![[S3_image 6.png]]

## S3 Glacier Flexible Retrieval

O S3 Glacier Flexible Retrieval oferece armazenamento de baixo custo, custo até 10% menor do que o S3 Glacier Instant Retrieval, para dados de arquivos que são acessados 1 a 2 vezes por ano e recuperados de forma assíncrona.

Suporte a SSL para dados em trânsito e criptografia de dados em repouso.

Tempos de recuperação configuráveis, de minutos a horas, com recuperações em massa gratuitas.

### Como funciona?

![[S3_image 7.png]]

## S3 Glacier Deep Archive

O S3 Glacier Deep Archive é uma classe de armazenamento mais acessível do Amazon S3 oferece suporte à retenção e preservação digital de longo prazo para dados que podem ser acessados uma ou duas vezes por ano.

Essa classe é projetada para clientes que mantêm conjuntos de dados por 7 a 10 anos ou mais para cumprir requisitos de conformidade regulatória.

### Como funciona?

![[S3_image 8.png]]

## S3 Outposts

O Amazon S3 on Outposts oferece armazenamento de objetos para seu ambiente de AWS Ouposts on-premises, ou seja, teremos um rack com os dados do Amazon S3 diretamente em nosso ambiente.

Projetado para armazenar dados de maneira duradoura e redundante em seus Outposts, ou seja, onde definirmos dentro do ambiente on-premisses.

A classe de armazenamento S3 Outposts é ideal para workloads com requisitos de residência de dados locais e para satisfazer as necessidades de desempenho exigentes, matendo os dados próximos às aplicações on-premises.

### Como funciona?

![[S3_image 9.png]]

# Criptografia

Antes de tudo, vamos explicar o que é criptografia. Criptografia é a prática de proteger informações por meio do uso de algoritmos codificados, hashes e assinaturas. As informações podem estar em repouso (com um arquivo em um HD), em trânsito (como comunicação eletrônica trocada entre duas ou mais partes) ou em uso (durante a computação de dados).

Por exemplo:

![[S3_image 10.png]]

<aside>
☝🏽

Um ponto que devo ressaltar é que desde 5 de janeiro de 2023, todos os novos uploads de objetos para o Amazon S3 são automaticamente criptografados (Amazon S3(SSE-S3)) sem custo adicional e sem impacto no desempenho.

</aside>

## Tipos de criptografia dentro da AWS

![[S3_image 11.png]]

### SSE-S3

Todos os uploads de novos objetos para buckets do S3 são criptografados por padrão com criptografia do lado do servidor com chaves gerenciadas pelo Amazon S3 (SSE-S3).

Este tipo de criptografia usa o Advanced Encryption Standard Galois/Counter Mode (AES-GCM) de 256 bits para criptografar todos os objetos carregados. Um ponto a ser lembrado, o Amazon S3 irá criptografar em repouso usando um chave exclusiva para cada objeto, e também teremos uma camada extra de proteção que é uma chave-mestra para acessarmos a chave exclusiva.

Sempre que formos criptografar um objeto ou visualizar um objeto criptografado, o mesmo terá o seguinte cabeçalho:

**“s3:x-amz-server-side-encryption”: “AES256”**

### Como funciona?

![[S3_image 12.png]]

### SSE-KMS

Ela é uma criptografia server-side em repouso (semelhante ao SSE-S3) com chaves gerenciadas pelo KMS (Key Management Service). 

O **AWS KMS** é um serviço que combina hardware e software seguros e altamente disponíveis para fornecer um sistema de gerenciamento de chaves escalado para a nuvem.

![[S3_image 13.png]]

Se usarmos as chaves do KMS, conseguimos:

- Criar, visualizar, editar, monitor, ativar/desativar, rotacionar e programar a exclusão de chaves KMS, tudo de forma centralizada;
- Definir políticas que controlam como e por quem as chaves do KMS podem ser usadas;
- Auditar o uso para provar que elas estão sendo usado corretamente;
- Rastreabilidade de acessos.

Tudo isso sem custo adicional por usar o **AWS KMS keys**.

O cabeçalho dessa criptografia é a seguinte:
**“s3:x-amz-server-side-encryption”: “aws:kms”**

### Como funciona?

![[S3_image 14.png]]

### DSSE-KMS

Essa criptografia é basicamente uma camada dupla de criptografia em repouso utilizando duas chaves KMS. Ela trabalha de forma semelhante à anterior, porém utiliza duas chaves do KMS.

Ao utilizarmos essa criptografia, devemos estar certos que as duas chaves devem estar na mesma região que o bucket.

O cabeçalho dessa criptografia é bem parecido com a padrão:
**“s3:x-amz-server-side-encryption”: “aws:kms:dsse”**

<aside>
☝🏽

Um detalhe legal, é que essa criptografia é foi lançada em junho de 2023 para atender as necessidades de compliance com a **National Security Agency CNSSP 15 para FIPS**, e **Data-at-Rest Capability Package (DAR CP) 5.0**.

</aside>

Nessa criptografia, ja temos um custo adicional para usá-la e também as chaves AWS KMS.

### SSE-C

Esta é uma criptografia server-side porém com chaves gerenciadas pelo cliente e não mais a AWS.

A criptografia de servidor envolve a proteção de dados em repouso. A criptografia no lado do servidor criptografa somente as dados de objetos, não os metadados de objeto.

Ao usarmos a criptografia do lado do servidro com chaves fornecidas pelo cliente (SSE-C), podemos armazenar dados criptografados com nossas próprias chaves de criptografia, o que deixa a responsabilidade toda para o cliente.

Não há cobranças adicionais pelo uso de SSE-C. No entanto, as solicitações para configurar e usar a SSE-C incorrem em cobranças padrão de solicitação Amazon S3.

O cabeçalho dessa criptografia é a seguinte:
**“s3:x-amz-server-side-encryption-customer-algorithm”: “true”**

### Como funciona?

![[S3_image 15.png]]

### Client-Side

Criptografia realizado no lado do cliente, antes de enviar dados para o Amazon S3, com isso as chaves são gerenciadas pelo próprio cliente.

A criptografia do lado do cliente é o ato de criptografar dados localmente para ajudar a garantir sua segurança em trânsito e em repouso.

### Como funciona?

![[S3_image 16.png]]