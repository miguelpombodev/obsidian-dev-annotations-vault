# O objetivo deste guia

Este guia não tem como objetivo ensinar apenas comandos do `kubectl`.

O objetivo é fazer com que, ao final, você consiga responder perguntas como:

- O que realmente acontece quando executo `kubectl apply`?
- Como um Pod é criado?
- Quem decide em qual máquina um Pod vai rodar?
- Como o Kubernetes detecta que um container morreu?
- Como funciona o balanceamento de carga?
- Como ocorre um Rolling Update sem downtime?
- Por que o Pod recebeu outro IP?
- Como um Service encontra meus Pods?
- Onde ficam armazenados os objetos do cluster?
- Como funciona o DNS interno?
- Como um Deployment sabe quantos Pods existem?
- Como o HPA sabe quando escalar?

Essas perguntas representam o conhecimento que diferencia alguém que sabe usar Kubernetes de alguém que entende sua arquitetura.

---

# O que você já sabe

Pelo seu contexto, você já domina boa parte da base necessária.

Você já entende:

- Containers
- Docker
- Docker Compose
- Imagens
- Volumes
- Redes Docker
- Linux
- Deploy de aplicações
- Cloud (EC2/Web Apps)

Isso elimina uma grande quantidade de conceitos introdutórios.

Nosso foco será entender **o sistema distribuído** que existe por trás do Kubernetes.

---

# Pré-requisitos conceituais

Antes de falar de Kubernetes, vale revisar rapidamente alguns conceitos.

## Processo

Um container nada mais é do que um processo isolado.

No Linux:

```
python app.py
```

é apenas um processo.

O Docker cria isolamento utilizando recursos do kernel como:

- namespaces
- cgroups
- capabilities
- seccomp
- OverlayFS

Ou seja...

O container **não é uma máquina virtual**.

É um processo isolado.

---

## Container

Um container é:

```
Filesystem

+

Processo

+

Configuração

+

Isolamento
```

Exemplo:

```
nginx

↓

Processo PID 1

↓

porta 80

↓

filesystem

↓

namespace de rede
```

---

## Máquina

Imagine um servidor Linux.

```
CPU

RAM

SSD

Rede
```

Nele você roda:

```
Docker

↓

Container A

Container B

Container C
```

Tudo simples.

Enquanto existir apenas uma máquina.

---

# O primeiro grande problema

Imagine uma API.

```
api

↓

Docker
```

Você publica a imagem.

Depois:

```
docker run api
```

Funciona perfeitamente.

Agora imagine que sua aplicação cresce.

Você precisa de:

```
10 containers
```

Ainda é simples.

```
docker compose up --scale api=10
```

Mas agora imagine:

```
3 servidores
```

Quem decide onde cada container ficará?

Docker Compose não resolve isso.

---

Agora imagine:

```
Servidor 1

↓

morreu
```

Quem recria os containers?

Docker Compose não resolve isso.

---

Agora imagine:

```
Atualizar versão

v1

↓

v2

↓

sem downtime
```

Quem controla isso?

Docker Compose também não resolve.

---

Agora imagine:

```
CPU > 80%

Criar mais containers
```

Quem monitora?

Quem cria?

Quem remove?

---

Agora imagine:

```
200 containers

50 máquinas

2 regiões

5 ambientes

500 deploys por dia
```

Nesse momento você não está mais administrando containers.

Você está administrando um sistema distribuído.

É exatamente aqui que nasce o Kubernetes.

---

# A evolução da infraestrutura

É interessante perceber a evolução natural.

## Era 1

Aplicações diretamente na máquina.

```
Servidor

↓

Nginx

↓

Python

↓

MySQL
```

Problemas:

- conflito de bibliotecas
- difícil atualização
- rollback complicado

---

## Era 2

Máquinas virtuais.

```
Hypervisor

↓

VM

↓

Ubuntu

↓

Python
```

Melhor isolamento.

Mas:

- pesado
- lento
- desperdício de recursos

---

## Era 3

Containers.

```
Docker

↓

Containers
```

Excelente.

Mas ainda falta coordenação.

---

## Era 4

Orquestração.

```
Docker

+

Kubernetes
```

Agora temos:

- escalabilidade
- auto-recuperação
- descoberta de serviços
- deploy declarativo
- balanceamento
- alta disponibilidade

---

# Kubernetes não substitui Docker

Esse é um erro muito comum.

Muitas pessoas dizem:

> Kubernetes substituiu Docker.

Isso não é correto.

Historicamente, o Kubernetes utilizava o Docker Engine como runtime.

Hoje ele utiliza uma interface chamada **CRI (Container Runtime Interface)**.

Na prática, normalmente utiliza:

- containerd
- CRI-O

Mas continua executando containers OCI.

A imagem continua sendo:

```
Dockerfile

↓

docker build

↓

OCI Image
```

Ou seja:

Docker continua extremamente importante.

O Kubernetes apenas não depende mais do Docker Engine para executar as imagens.

---

# O verdadeiro problema que Kubernetes resolve

Essa talvez seja a ideia mais importante de todo o guia.

Docker executa containers.

Kubernetes mantém um estado.

Essa diferença parece pequena.

Mas muda completamente a forma de pensar.

Docker:

```
Execute este container.
```

Kubernetes:

```
Garanta que sempre existam

5 containers

rodando

com esta configuração.
```

Perceba a diferença.

Docker é imperativo.

```
Faça isso.
```

Kubernetes é declarativo.

```
Este é o estado desejado.
```

Essa única mudança explica praticamente toda a arquitetura do Kubernetes.

---

# O conceito de Estado Desejado (Desired State)

Imagine um arquivo YAML.

```
replicas: 3

image: api:v1
```

Você envia isso ao cluster.

Você não disse:

```
Crie um Pod.

Depois outro.

Depois outro.
```

Você apenas declarou:

```
Quero três Pods.
```

Quem fará o trabalho?

O Kubernetes.

Se um morrer:

```
3

↓

2

↓

Kubernetes cria outro

↓

3
```

Você não precisa intervir.

Isso é reconciliação de estado.

É a base de toda a arquitetura.

---

# A analogia do "termostato"

Imagine um ar-condicionado configurado para 22 °C.

O termostato não liga o compressor apenas uma vez.

Ele mede continuamente.

```
Temperatura atual

↓

20°C

↓

Desliga
```

Depois:

```
24°C

↓

Liga novamente
```

Ele compara constantemente:

```
Estado atual

VS

Estado desejado
```

Kubernetes faz exatamente isso.

```
Desejado:

5 Pods

↓

Atual:

4 Pods

↓

Criar 1 Pod
```

Ou:

```
Desejado:

5 Pods

↓

Atual:

6 Pods

↓

Remover 1
```

Essa lógica é implementada pelos **Controllers**, que estudaremos em detalhes mais adiante.

---

# Mental Model

A partir de agora, tente abandonar a mentalidade de "executar containers" e adotar a de "descrever sistemas".

Você não gerencia processos individualmente; você descreve um estado desejado, e o cluster trabalha continuamente para convergir o estado real ao estado declarado.

Esse modelo mental será recorrente em praticamente todos os recursos do Kubernetes: Deployments, ReplicaSets, Services, ConfigMaps, Secrets, Horizontal Pod Autoscaler, Jobs e outros.

---

# Resumo do capítulo

- Kubernetes é um orquestrador de containers baseado em estado declarativo.
- Ele resolve problemas de operação em ambientes distribuídos, não de execução local de containers.
- O conceito central é **Desired State** (estado desejado).
- Controllers monitoram continuamente a diferença entre o estado desejado e o estado atual.
- O Kubernetes não substitui os containers; ele coordena sua execução em um cluster.