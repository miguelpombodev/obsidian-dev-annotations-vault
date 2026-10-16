---
tags:
  - kubernetes
  - deployment
  - rollout
  - rollback
  - rolling-update
---
> [!INFO] Objetivo
> Entender o Deployment como o principal controlador de aplicações stateless, compreender sua relação com ReplicaSets e aprender como o Kubernetes realiza atualizações de aplicações sem indisponibilidade.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender o que é um Deployment.
- [ ] Explicar a relação entre Deployment, ReplicaSet e Pods.
- [ ] Entender como funciona um Rolling Update.
- [ ] Realizar Rollbacks.
- [ ] Alterar imagens de aplicações.
- [ ] Compreender as estratégias de atualização.

---

# Pré-requisitos

- [[02 - Arquitetura do Kubernetes]]
- [[03 - Objetos da API]]
- [[04 - Pods]]
- [[05 - Labels, Selectors e Annotations]]
- [[06 - ReplicaSets]]

---

# O problema que este capítulo resolve

Imagine que sua aplicação possui três Pods.

```text
Pod v1

Pod v1

Pod v1
```

Agora uma nova versão precisa ser publicada.

Como atualizar todos os Pods?

Como evitar downtime?

Como desfazer rapidamente uma atualização problemática?

O ReplicaSet não resolve esse problema.

É exatamente para isso que existe o Deployment.

---

# O que é um Deployment?

Um Deployment é um Controller responsável por gerenciar o ciclo de vida de uma aplicação.

Mas atenção.

Ele **não gerencia Pods diretamente**.

Seu relacionamento é indireto.

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Essa camada adicional permite implementar estratégias sofisticadas de atualização.

---

# Modelo Mental

Pense em um Deployment como um gerente de versões.

Ele não executa funcionários (Pods).

Ele gerencia equipes (ReplicaSets).

Cada equipe é responsável por manter seus próprios funcionários.

---

# Arquitetura

```text
Deployment

↓

ReplicaSet

↓

Pod

↓

Container
```

Essa hierarquia é fundamental.

---

# Anatomia de um Deployment

Exemplo mínimo.

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx

spec:

  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:

      labels:

        app: nginx

    spec:

      containers:

      - name: nginx

        image: nginx:1.25
```

Observe que a estrutura é praticamente igual ao ReplicaSet.

A grande diferença é o comportamento interno.

---

# Como um Deployment funciona?

Quando aplicamos esse manifesto:

```bash
kubectl apply -f deployment.yaml
```

O Deployment **não cria Pods**.

Ele cria um ReplicaSet.

```text
Deployment

↓

ReplicaSet
```

O ReplicaSet então cria os Pods.

```text
ReplicaSet

↓

Pods
```

Fluxo completo.

```text
Deployment

↓

ReplicaSet

↓

Pods

↓

Containers
```

---

# Internals

Quando um Deployment é criado.

O fluxo é:

```text
API Server

↓

Deployment Controller

↓

Criar ReplicaSet

↓

ReplicaSet Controller

↓

Criar Pods

↓

Scheduler

↓

Worker Node

↓

Container Runtime

↓

Containers
```

Observe a quantidade de Controllers envolvidos.

---

# O primeiro ReplicaSet

Após criar um Deployment.

Podemos listar.

```bash
kubectl get deployments
```

Depois.

```bash
kubectl get rs
```

Resultado.

```text
NAME

nginx-67b984f8c
```

Depois.

```bash
kubectl get pods
```

```text
nginx-67b984f8c-xxxxx

nginx-67b984f8c-yyyyy

nginx-67b984f8c-zzzzz
```

Perceba que o nome dos Pods contém o nome do ReplicaSet.

---

# Atualizando uma aplicação

Imagine.

Versão atual.

```yaml
image: nginx:1.25
```

Agora alteramos.

```yaml
image: nginx:1.26
```

Aplicamos novamente.

```bash
kubectl apply -f deployment.yaml
```

O Deployment detecta que houve alteração no Pod Template.

Ele cria um novo ReplicaSet.

---

# Rolling Update

Fluxo.

Estado inicial.

```text
ReplicaSet A

↓

Pod v1

Pod v1

Pod v1
```

Novo Deployment.

```text
ReplicaSet B

↓

Pod v2
```

Agora.

```text
ReplicaSet A

3

↓

2

↓

1

↓

0
```

Enquanto.

```text
ReplicaSet B

0

↓

1

↓

2

↓

3
```

Nunca existe indisponibilidade.

Esse processo é chamado de **Rolling Update**.

---

# Por que criar outro ReplicaSet?

Essa é uma das perguntas mais importantes.

O Deployment não altera o ReplicaSet existente.

Ele cria um novo.

Assim temos.

```text
ReplicaSet A

↓

v1
```

```text
ReplicaSet B

↓

v2
```

Isso permite voltar para a versão anterior.

---

# Rollback

Imagine.

Nova versão.

```text
v2
```

Algo deu errado.

Executamos.

```bash
kubectl rollout undo deployment nginx
```

O Deployment reduz o ReplicaSet novo.

E aumenta novamente o ReplicaSet anterior.

---

# Histórico

Ver histórico.

```bash
kubectl rollout history deployment nginx
```

Resultado.

```text
REVISION

1

2

3
```

Cada revisão corresponde a um ReplicaSet.

---

# Verificando atualização

Acompanhar.

```bash
kubectl rollout status deployment nginx
```

Muito utilizado em pipelines CI/CD.

---

# Estratégias de atualização

Existem duas principais.

## RollingUpdate

Padrão.

Atualização gradual.

Sem downtime.

---

## Recreate

Todos os Pods antigos são removidos.

Depois.

Os novos são criados.

```text
Remover

↓

Criar
```

Existe indisponibilidade.

---

# maxUnavailable

Controla quantos Pods podem ficar indisponíveis.

Exemplo.

```yaml
strategy:

  rollingUpdate:

    maxUnavailable: 1
```

Significa.

Durante a atualização.

No máximo um Pod poderá ficar indisponível.

---

# maxSurge

Controla quantos Pods extras podem existir.

```yaml
maxSurge: 1
```

Imagine.

```text
Replicas

3
```

Durante o deploy.

```text
4 Pods
```

Um Pod extra é criado temporariamente.

Isso reduz indisponibilidade.

---

# Atualizando imagem

Também podemos utilizar.

```bash
kubectl set image deployment/nginx nginx=nginx:1.26
```

Depois.

```bash
kubectl rollout status deployment nginx
```

---

# Hands-on

Criar Deployment.

```bash
kubectl apply -f deployment.yaml
```

Ver.

```bash
kubectl get deployment
```

ReplicaSets.

```bash
kubectl get rs
```

Pods.

```bash
kubectl get pods
```

Alterar imagem.

```yaml
image: nginx:1.26
```

Aplicar novamente.

```bash
kubectl apply -f deployment.yaml
```

Acompanhar.

```bash
kubectl get rs -w
```

Observe.

Um ReplicaSet cresce.

Outro diminui.

---

# Como isso aparece na prática

Praticamente todas as aplicações stateless em Kubernetes utilizam Deployments.

Exemplos.

- APIs REST
- Aplicações Django
- FastAPI
- Flask
- Node.js
- Java Spring Boot
- Go
- Nginx
- Frontends React
- Vue
- Angular

Sempre que pensamos em uma aplicação web sem estado persistente, normalmente pensamos em um Deployment.

---

# Relação com os capítulos anteriores

Até agora a arquitetura completa ficou assim.

```text
Deployment

↓

ReplicaSet

↓

Selector

↓

Labels

↓

Pods

↓

Containers
```

Esse diagrama continuará crescendo ao longo do guia.

---

# Boas práticas

- Utilize Deployments para aplicações stateless.
- Nunca altere Pods manualmente.
- Atualize sempre o Deployment.
- Utilize RollingUpdate como estratégia padrão.
- Monitore os Rollouts durante deploys.

---

# Erros comuns

> [!WARNING]

Editar um Pod diretamente.

Na próxima reconciliação ele será recriado pelo ReplicaSet.

---

> [!WARNING]

Confundir Deployment com ReplicaSet.

O Deployment gerencia ReplicaSets.

O ReplicaSet gerencia Pods.

---

> [!WARNING]

Utilizar Recreate sem necessidade.

Na maioria das aplicações web isso gera downtime.

---

# Resumo

Neste capítulo aprendemos que:

- Deployments gerenciam ReplicaSets.
- ReplicaSets gerenciam Pods.
- Atualizações criam novos ReplicaSets.
- Rolling Updates permitem deploys sem indisponibilidade.
- Rollbacks reutilizam ReplicaSets antigos.
- O histórico de versões é mantido pelo Deployment.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O Deployment cria Pods diretamente?
- [ ] Por que um novo ReplicaSet é criado durante uma atualização?
- [ ] O que é um Rolling Update?
- [ ] Como realizar um Rollback?
- [ ] Qual a diferença entre RollingUpdate e Recreate?
- [ ] Como acompanhar um rollout?

---

# Diagrama consolidado

Até aqui, a arquitetura completa que estudamos é:

```text
                Deployment
                      │
                      ▼
                 ReplicaSet
                      │
               matchLabels
                      │
                      ▼
                   Labels
                      │
                      ▼
                     Pods
                 ┌────┴────┐
                 │         │
         Container A   Container B
                      │
               Shared Network
               Shared Volumes
```
