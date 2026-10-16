---
tags:
  - kubernetes
  - pods
  - containers
  - namespaces
---
> [!INFO] Objetivo
> Entender o que é um Pod, por que ele existe, como funciona internamente e por que ele é a menor unidade de agendamento do Kubernetes.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar por que Pods existem.
- [ ] Entender por que um Pod não é um container.
- [ ] Explicar como containers compartilham recursos dentro de um Pod.
- [ ] Entender o ciclo de vida de um Pod.
- [ ] Criar e inspecionar Pods.
- [ ] Entender por que Pods são efêmeros.

---

# Pré-requisitos

- [[01 - Por que Kubernetes existe]]
- [[02 - Arquitetura do Kubernetes]]
- [[03 - Objetos da API]]

---

# O problema que este capítulo resolve

Quando aprendemos Docker, normalmente pensamos:

```text
Aplicação

↓

Container
```

Então surge uma dúvida natural.

> Se o Kubernetes trabalha com containers, por que ele criou outra abstração chamada **Pod**?

Por que não agendar containers diretamente?

Essa decisão foi arquitetural e resolve diversos problemas.

---

# O que é um Pod?

Um Pod é **a menor unidade de execução e agendamento do Kubernetes**.

Mas essa definição, sozinha, não ajuda muito.

Uma definição mais útil seria:

> Um Pod é um grupo de um ou mais containers que compartilham recursos e possuem o mesmo ciclo de vida.

Visualmente:

```text
             Pod

     ┌─────────────────────┐

     │  Container A        │

     │                     │

     │  Container B        │

     └─────────────────────┘
```

O Scheduler nunca agenda um container.

Ele agenda um **Pod**.

---

# Por que o Kubernetes criou os Pods?

Imagine que o Kubernetes agendasse containers individualmente.

Teríamos problemas como:

- Como garantir que dois containers iniciem juntos?
- Como compartilhar armazenamento?
- Como compartilhar rede?
- Como garantir que ambos sejam removidos juntos?

O Pod resolve exatamente isso.

Ele cria um limite lógico em torno de um conjunto de containers.

---

# Um Pod geralmente possui apenas um container

Embora um Pod possa conter vários containers, na maioria dos casos encontramos:

```text
Pod

↓

Container
```

Ou seja:

```text
1 Pod

↓

1 Container
```

Isso é o cenário mais comum.

> [!TIP]
> Não pense que um Pod deve conter vários containers.
>
> A maioria dos Pods em produção possui apenas um.

---

# Quando utilizar múltiplos containers?

Existem situações onde faz sentido.

Exemplo clássico:

```text
Pod

├── API

└── Log Collector
```

Ou

```text
Pod

├── Aplicação

└── Proxy
```

Esse padrão é chamado de **Sidecar Pattern**.

Voltaremos a ele em capítulos futuros.

---

# O que os containers compartilham?

Todos os containers de um mesmo Pod compartilham alguns recursos.

Principalmente:

- Network Namespace
- IPC Namespace
- Volumes
- Hostname

---

## Compartilhando a rede

Este é provavelmente o conceito mais importante.

Todos os containers do mesmo Pod possuem o mesmo endereço IP.

Exemplo.

```text
Pod

IP

10.244.0.15
```

Dentro dele:

```text
Container A

10.244.0.15
```

```text
Container B

10.244.0.15
```

Eles compartilham o mesmo Network Namespace.

Consequentemente:

```text
localhost
```

é compartilhado.

Isso significa que um container pode acessar outro utilizando:

```text
http://localhost
```

Sem passar pela rede do cluster.

---

# Internals — Network Namespace

Esse comportamento é herdado diretamente do Linux.

O Kubernetes utiliza Linux Namespaces.

Quando o Pod é criado:

```text
Criar Network Namespace

↓

Criar interface de rede

↓

Associar IP

↓

Criar containers

↓

Todos entram no mesmo Namespace
```

É por isso que eles compartilham:

- IP
- portas
- localhost

---

# Compartilhando Volumes

Containers também podem compartilhar armazenamento.

```text
Pod

↓

Volume

↓

Container A

↓

Container B
```

Isso facilita:

- troca de arquivos;
- cache;
- logs;
- sockets Unix.

---

# Compartilhando ciclo de vida

Todos os containers do Pod nascem juntos.

```text
Criar Pod

↓

Criar todos os containers
```

E morrem juntos.

```text
Excluir Pod

↓

Todos os containers removidos
```

Não existe "metade" de um Pod.

---

# Pods são efêmeros

Esse é um conceito extremamente importante.

Um Pod não foi feito para durar.

Ele pode desaparecer por diversos motivos.

Exemplo:

- Node reiniciou.
- Atualização.
- Escalabilidade.
- Falha.
- Eviction.
- Rolling Update.

Quando isso acontece:

```text
Pod antigo

↓

Removido

↓

Novo Pod criado
```

O novo Pod é outro objeto.

Mesmo que tenha exatamente a mesma configuração.

---

# Nunca dependa do IP de um Pod

Como Pods são efêmeros:

Hoje:

```text
10.244.1.15
```

Depois de um restart:

```text
10.244.3.42
```

Depois:

```text
10.244.8.10
```

O IP muda.

Por isso utilizamos Services.

---

# Anatomia de um Pod

Manifesto mínimo.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest
```

---

# Explicando o YAML

## apiVersion

Estamos utilizando a API Core v1.

```yaml
apiVersion: v1
```

---

## kind

Estamos criando um Pod.

```yaml
kind: Pod
```

---

## metadata

Nome do Pod.

```yaml
metadata:
  name: nginx
```

---

## spec

Descrição do estado desejado.

---

## containers

Lista de containers.

```yaml
containers:
```

Mesmo quando existe apenas um.

O campo continua sendo uma lista.

---

## image

Imagem Docker.

```yaml
image: nginx:latest
```

---

# Internals — Como um Pod nasce?

Fluxo completo.

```text
kubectl apply

↓

API Server

↓

etcd

↓

Scheduler

↓

Escolha do Node

↓

kubelet

↓

containerd

↓

Pull da imagem

↓

Criar Network Namespace

↓

Criar Volumes

↓

Criar Containers

↓

Pod Running
```

---

# Estados de um Pod

Durante sua vida um Pod passa por diferentes fases.

```text
Pending

↓

Running

↓

Succeeded
```

Ou

```text
Pending

↓

Running

↓

Failed
```

Ou ainda

```text
Pending

↓

Running

↓

Unknown
```

Os estados mais comuns são:

| Estado | Significado |
|---------|-------------|
| Pending | Ainda não iniciou completamente |
| Running | Executando normalmente |
| Succeeded | Finalizado com sucesso |
| Failed | Finalizado com erro |
| Unknown | Estado desconhecido |

---

# Comandos úteis

Criar.

```bash
kubectl apply -f pod.yaml
```

Listar.

```bash
kubectl get pods
```

Detalhes.

```bash
kubectl describe pod nginx
```

Logs.

```bash
kubectl logs nginx
```

Executar comando.

```bash
kubectl exec -it nginx -- sh
```

Remover.

```bash
kubectl delete pod nginx
```

---

# Hands-on

## Criando um Pod

Arquivo:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest
```

Aplicar:

```bash
kubectl apply -f pod.yaml
```

Verificar:

```bash
kubectl get pods
```

Observar detalhes:

```bash
kubectl describe pod nginx
```

Entrar no container:

```bash
kubectl exec -it nginx -- sh
```

---

# Como isso aparece na prática

Na prática, quase nunca criamos Pods diretamente.

Quem cria Pods normalmente são:

- Deployment
- ReplicaSet
- DaemonSet
- Job
- StatefulSet

Criar Pods manualmente é útil para:

- testes;
- depuração;
- aprendizado.

Em produção, normalmente utilizamos controladores.

---

# Boas práticas

- Utilize um único container por Pod, salvo quando houver um motivo claro.
- Nunca dependa do IP do Pod.
- Nunca armazene dados importantes dentro do sistema de arquivos do container.
- Considere o Pod como um recurso descartável.

---

# Erros comuns

> [!WARNING]
> Acreditar que Pod é sinônimo de container.

Um Pod pode conter vários containers.

---

> [!WARNING]
> Utilizar Pods diretamente em produção.

O recomendado é utilizar Deployments ou outros controladores.

---

> [!WARNING]
> Armazenar estado dentro do Pod.

Pods podem ser recriados a qualquer momento.

---

# Resumo

Neste capítulo aprendemos que:

- O Pod é a menor unidade de agendamento do Kubernetes.
- Um Pod encapsula um ou mais containers.
- Containers do mesmo Pod compartilham rede e armazenamento.
- Todos os containers compartilham o mesmo ciclo de vida.
- Pods são efêmeros.
- Nunca devemos depender do IP de um Pod.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que o Kubernetes criou Pods?
- [ ] Um Pod é um container?
- [ ] O que os containers compartilham dentro de um Pod?
- [ ] Por que Pods são efêmeros?
- [ ] Quando faz sentido utilizar múltiplos containers?
- [ ] Por que não devemos depender do IP de um Pod?
