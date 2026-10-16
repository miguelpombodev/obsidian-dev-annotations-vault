---
tags:
  - api
  - comunication
  - grpc
  - cncf
---
É uma framework desenvolvido pela Google que tem o objetivo de facilitar processo de comunicação entre sistemas de uma forma extremamente rápida, leve e independente a linguagem.
E apesar do gRPC ter sido desenvolvido pela Google, o projeto é mantido pela CNCF (Cloud Native Computing Foundation)

## Quando usar o gRPC?
Usamos majoritariamente para comunicação entre microserviços por conta de estarmos trafegando as mensagens entre os serviços o tempo todo então a leveza, a rapidez e a segurança do gRPC são pontos-chave aqui.

**Conseguimos usar  também com**:
- Mobile - se comunicando de forma direta;
- Browsers (através do projeto gRPC Web, porém este projeto ainda não é maduro o bastante);
- Backend;
- Geração das bibliotecas de forma automática;
- Streaming bidirecional utilizando HTTP/2;

> [!INFO] HTTP/2
> 	Este protocolo é importante pois conseguimos trafegar dados em binário e consegue fazer comunicações bidirecionais, ou seja, conseguimos abrir um canal de comunicação e com isso recebemos diversas "responses" e enviamos diversos "requests" de forma contínua 

## Linguagens com suporte oficial
- gRPC-GO
- gRPC-JAVA
	- Kotlin
- gRPC-C
	- C++
	- Python
	- Objective C
	- PHP 
	- C#
	- Node.js
	- Dart

## RPC - Remote Procedure Call
É um protocolo que permite a comunicação entre diferentes sistemas ou partes de um software que estão em máquinas diferentes. Em outras palavras, o RPC permite que um sistema “chame” uma função ou procedimento em outro sistema como se estivesse local, mesmo que estejam em locais remotos. Essa abordagem é bastante útil em **sistemas distribuídos**, já que diferentes partes do sistema precisam se comunicar e trocar dados de forma rápida e eficiente.

![[Pasted image 20260727132515.png]]

Antigamente fazia-se muito este uso via XML por conta dos contratos pré-definidos e tudo mais.

## HTTP/2
O HTTP/2 é um protocolo criado pela Google que primariamente teve o nome de SPDY (_Speedy_), lançado em 2015, e como dissemos antes, os dados nesse protocolo são trafegado em binários e não em ***"raw text"*** como no HTTP/1.

Este protocolo utiliza a conexão TCP para enviar e receber dados do cliente e do servidor (Multiplex).

> [!INFO] Multiplex
> Um **servidor multiplex** é um sistema que gere e combina vários fluxos de dados ou ligações de rede numa única linha de comunicação. As principais categorias que definem este conceito incluem:
> - **E/S de Redes (I/O Multiplexing):** Um programa de servidor que utiliza funções do sistema operativo (como `select` ou `poll`) para vigiar e processar pedidos de múltiplos clientes em simultâneo através de uma única thread ou processo. 
> - **Protocolos de Camada de Aplicação:** Tecnologias como o HTTP/2 que enviam múltiplos pedidos e respostas em paralelo usando uma única conexão TCP física. 
> - **Arquiteturas de Bases de Dados:** Sistemas de alto desempenho (como o SAP IQ) onde múltiplos servidores partilham o acesso a dados de forma coordenada. 

Também temos os recursos de **Server Push**, que basicamente conseguimos mandar diversas informações para o cliente como arquivos estáticos.

E finalmente, os headers do HTTP/2 são comprimidos por padrão.

## Protocol Buffers ou Protobuf
Como o gRPC usa o HTTP/2 para o tráfego dos dados e para o este tráfego utilizamos o tipo de protocolo chamados **Protocol Buffers**.

Os Protocol Buffers são uma linguagem e plataforma neutral, com mecanismos extensíveis e serialização de dados criado pela Google e são **Open-Source**. Pense como um XML porém menor, mais rápido e mais simples. 
O pulo do gato aqui é que ele trabalha com esses dados de forma binária e que precisamos de um contrato definido (**schema chamado de protofile**) porém mesmo assim precisamos serializar e deserializar os dados indo e vindo, diferente do JSON ou XML que o parsing dos dados requerem mais CPU para este processo.

![[Pasted image 20260727134315.png]]

Fonte : [Google sobre Protocol  Buffers](https://protobuf.dev/)

## Protocol Buffers vs JSON

Aqui estão as diferenças entre a utilização dos Protobufs e JSON:
- Arquivos binários (Bin files) são mais leves arquivos JSON;
- O processo de serializacão é mais leve do que JSON
- Gasta menos recursos de rede;
- Processo é mais veloz;

## Formatos de tráfego e comunicação
Nesta seção, teremos listados as forma de comunicação que temos as condições de fazer ao utilizar o HTTP/2 e gRPC.

### gRPC API "unary"
Este formato é um que já estamos bem acostumados, como por exemplo com comunicações em **API RESTs**, que basicamente é a comunicação server-client, onde o client envia um request e recebe uma responsa do servidor.

![[Pasted image 20260727160356.png]]

### gRPC API "Server Streaming"
Este formato seguirá o mesmo formato que o anterior, teremos a arquitetura cliente-servidor, porém agora o cliente enviará uma request e pode receber diversas responses do mesmo servidor.

Este formato nos ajuda a ter os dados de um processo pesado, por exemplo se nesta request enviada precisarmos de uma "penca" de informações, invés de esperar uma única resposta, na qual pode demorar pode conta da quantidade e/ou do tamanho dos dados retornados, "splitamos" esta response em várias outras.

![[Pasted image 20260727160629.png]]

### gRPC - API "Client Streaming"
Agora teremos o oposto do "Server Streaming", pensemos que no caso em nosso temos um arquivo gigantesco, invés de alocarmos todo o arquivo na memória do cliente e assim perdermos performance, o que podemos fazer é enviar partes deste arquivos (***chunks***) para o servidor assim que formos processando o mesmo em diversas request.

Lembrando que tudo isso é feito por conta do HTTP/2.

![[Pasted image 20260727161231.png]]


### gRPC - API "Bi-directional Streaming"
Aqui temos a junção do "Server Streaming" e "Client Streaming", ou seja, ambos o client e o server irão enviar os dados de forma contínua um para o outro.

![[Pasted image 20260727161424.png]]
