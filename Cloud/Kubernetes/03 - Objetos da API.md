---
tags:
  - kubernetes
  - api
  - yaml
  - manifests
---
> [!INFO] Objetivo
> Entender como o Kubernetes representa todos os seus recursos através da API, compreender a estrutura dos manifestos YAML e aprender como o API Server interpreta esses objetos.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender o que é um Objeto da API.
- [ ] Explicar a função de `apiVersion`.
- [ ] Explicar a função de `kind`.
- [ ] Entender `metadata`.
- [ ] Entender `spec`.
- [ ] Entender `status`.
- [ ] Diferenciar estado desejado e estado atual.
- [ ] Utilizar `kubectl explain`.

---

# Pré-requisitos

- [[01 - Por que Kubernetes existe]]
- [[02 - Arquitetura do Kubernetes]]

---

# O problema que este capítulo resolve

Nos capítulos anteriores vimos que o Kubernetes é um sistema declarativo.

Mas...

Como informamos ao Kubernetes o estado desejado?

A resposta é:

> **Através de Objetos da API.**

Tudo que existe dentro do Kubernetes é um objeto.

Por exemplo:

- Pod
- Deployment
- Service
- Secret
- ConfigMap
- Namespace
- PersistentVolume
- Job
- CronJob
- Ingress

Todos seguem exatamente a mesma estrutura.

---

# O Kubernetes é uma API

O nome Kubernetes pode dar a impressão de que estamos trabalhando com um software que cria containers.

Na realidade estamos interagindo com uma API REST.

Quando executamos:

```bash
kubectl apply -f deployment.yaml
```

O kubectl envia uma requisição HTTPS para o API Server.

```text
kubectl

↓

HTTPS

↓

API Server

↓

Objeto da API

↓

etcd
```

O YAML nunca é executado.

Ele apenas descreve um objeto.

---

# Todo recurso possui a mesma estrutura

Independentemente do recurso, a estrutura sempre será semelhante.

```yaml
apiVersion:
kind:
metadata:
spec:
status:
```

Essa é provavelmente a estrutura mais importante de todo Kubernetes.

---

# apiVersion

O primeiro campo informa qual versão da API será utilizada.

Exemplo:

```yaml
apiVersion: v1
```

Outro exemplo:

```yaml
apiVersion: apps/v1
```

Outro:

```yaml
apiVersion: networking.k8s.io/v1
```

---

## Por que existe apiVersion?

O Kubernetes evolui constantemente.

Imagine que um recurso foi criado em 2018.

Agora estamos em 2025.

A estrutura desse recurso provavelmente mudou.

Ao utilizar versões da API, o Kubernetes consegue manter compatibilidade.

Exemplo:

```text
apps/v1
```

```text
batch/v1
```

```text
autoscaling/v2
```

Cada grupo da API possui seu próprio ciclo de evolução.

---

# Como descobrir a apiVersion?

Utilizando:

```bash
kubectl api-resources
```

Ou:

```bash
kubectl api-versions
```

Esses dois comandos serão utilizados frequentemente.

---

# kind

O campo `kind` informa qual objeto estamos criando.

Exemplo:

```yaml
kind: Pod
```

Outro:

```yaml
kind: Deployment
```

Outro:

```yaml
kind: Service
```

O Kubernetes possui dezenas de tipos diferentes.

Todos são identificados por esse campo.

---

# metadata

O bloco `metadata` contém informações sobre o próprio objeto.

Exemplo:

```yaml
metadata:
  name: minha-api
```

Todo objeto possui metadados.

---

## Campos mais comuns

```yaml
metadata:
  name:
  namespace:
  labels:
  annotations:
```

Veremos Labels e Annotations em um capítulo específico.

---

### name

Nome único do objeto.

```yaml
metadata:
  name: api-pedidos
```

---

### namespace

Indica em qual Namespace o objeto será criado.

```yaml
metadata:
  namespace: production
```

Caso seja omitido:

```text
default
```

será utilizado.

---

### labels

Labels são pares chave/valor utilizados para organizar recursos.

```yaml
labels:
  app: pedidos
  environment: production
```

Elas são extremamente importantes.

Services, Deployments e ReplicaSets utilizam Labels para localizar Pods.

---

### annotations

Annotations também são pares chave/valor.

Mas diferentemente das Labels:

- não são utilizadas para seleção;
- normalmente armazenam metadados adicionais.

Exemplo:

```yaml
annotations:
  owner: backend-team
```

---

# spec

O campo `spec` representa o estado desejado.

É a parte mais importante do manifesto.

Exemplo:

```yaml
spec:
  replicas: 3
```

Estamos dizendo:

> Quero três réplicas.

Outro exemplo:

```yaml
spec:
  containers:
```

Outro:

```yaml
spec:
  ports:
```

Cada recurso possui sua própria especificação.

---

# status

O campo `status` representa o estado atual.

Esse campo normalmente **não é preenchido por nós**.

Ele é atualizado pelo Kubernetes.

Exemplo.

Estado desejado:

```yaml
spec:
  replicas: 3
```

Estado atual:

```yaml
status:
  replicas: 3
  readyReplicas: 2
```

Perceba que ainda falta uma réplica ficar pronta.

---

# Desired State x Current State

Esse é o coração do Kubernetes.

```text
spec

↓

Estado Desejado
```

```text
status

↓

Estado Atual
```

O Controller compara ambos.

```text
Spec

↓

Status

↓

Existe diferença?

↓

Corrigir
```

É exatamente esse processo que estudamos no capítulo anterior.

---

# Exemplo completo

Vamos analisar um manifesto simples.

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

Linha por linha.

---

## apiVersion

```yaml
apiVersion: v1
```

Estamos utilizando a API Core v1.

---

## kind

```yaml
kind: Pod
```

Estamos criando um Pod.

---

## metadata

```yaml
metadata:
  name: nginx
```

Nome do objeto.

---

## spec

```yaml
spec:
```

Início da descrição do estado desejado.

---

## containers

```yaml
containers:
```

Lista de containers do Pod.

Mais adiante veremos que um Pod pode possuir mais de um container.

---

## image

```yaml
image: nginx:latest
```

Imagem que será utilizada.

---

# Como o API Server interpreta um YAML

Quando executamos:

```bash
kubectl apply -f pod.yaml
```

O fluxo é:

```text
Ler arquivo YAML

↓

Converter para JSON

↓

Enviar via HTTPS

↓

API Server

↓

Validar Schema

↓

Persistir no etcd

↓

Controllers observam novo objeto

↓

Estado começa a ser reconciliado
```

Perceba que o YAML nunca chega aos Workers.

Ele existe apenas para facilitar nossa escrita.

Internamente a comunicação ocorre em JSON.

---

# kubectl explain

Uma das ferramentas mais úteis do Kubernetes.

Exemplo:

```bash
kubectl explain pod
```

Saída:

```text
DESCRIPTION:

Pod is a collection of containers...
```

Podemos aprofundar.

```bash
kubectl explain pod.spec
```

Mais específico.

```bash
kubectl explain pod.spec.containers
```

Esse comando consulta diretamente o schema da API.

É muito melhor do que procurar documentação durante o desenvolvimento.

---

# Descobrindo recursos

Para listar todos os recursos disponíveis.

```bash
kubectl api-resources
```

Exemplo.

```text
pods

deployments

services

jobs

cronjobs

configmaps

secrets

...
```

---

# Descobrindo versões

```bash
kubectl api-versions
```

Saída semelhante.

```text
apps/v1

batch/v1

networking.k8s.io/v1

autoscaling/v2

...
```

---

# Como isso aparece na prática

Imagine que um desenvolvedor cria um Deployment.

Na realidade ele não está "criando um Deployment".

Ele está enviando um objeto JSON para a API do Kubernetes.

O API Server:

- valida;
- persiste;
- notifica os Controllers.

A partir desse momento o cluster inteiro trabalha para transformar aquele documento YAML em uma aplicação executando.

---

# Boas práticas

- Sempre utilize versões estáveis da API.
- Nunca utilize APIs depreciadas em novos projetos.
- Utilize `kubectl explain` antes de recorrer ao Google.
- Pense em recursos como Objetos da API.
- Escreva manifestos declarativos.

---

# Erros comuns

> [!WARNING]
> Acreditar que YAML é uma linguagem do Kubernetes.

YAML é apenas um formato de serialização.

---

> [!WARNING]
> Pensar que `status` deve ser editado.

Ele é gerenciado automaticamente pelo cluster.

---

> [!WARNING]
> Confundir `metadata` com `spec`.

Metadata descreve o objeto.

Spec descreve o estado desejado.

---

# Resumo

Neste capítulo aprendemos que:

- Todo recurso do Kubernetes é um Objeto da API.
- Todos os objetos compartilham uma estrutura comum.
- `apiVersion` identifica a versão da API.
- `kind` identifica o tipo do recurso.
- `metadata` descreve o objeto.
- `spec` representa o estado desejado.
- `status` representa o estado atual.
- O API Server interpreta manifestos e inicia o processo de reconciliação.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que é um Objeto da API?
- [ ] Qual a função de `apiVersion`?
- [ ] Qual a função de `kind`?
- [ ] O que pertence ao `metadata`?
- [ ] Qual a diferença entre `spec` e `status`?
- [ ] O que faz o comando `kubectl explain`?
- [ ] O YAML é enviado diretamente para os Worker Nodes?
