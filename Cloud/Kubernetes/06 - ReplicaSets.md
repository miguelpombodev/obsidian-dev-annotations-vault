---
tags:
  - kubernetes
  - replicaset
  - controller
  - reconciliation
---
> [!INFO] Objetivo
> Entender o papel dos ReplicaSets no Kubernetes, como eles garantem a disponibilidade dos Pods e por que representam a primeira implementação prática do Reconciliation Loop.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender o que é um ReplicaSet.
- [ ] Explicar por que ele existe.
- [ ] Entender como funciona o processo de reconciliação.
- [ ] Explicar a relação entre ReplicaSets e Pods.
- [ ] Entender por que normalmente não criamos ReplicaSets manualmente.
- [ ] Inspecionar ReplicaSets utilizando kubectl.

---

# Pré-requisitos

- [[02 - Arquitetura do Kubernetes]]
- [[03 - Objetos da API]]
- [[04 - Pods]]
- [[05 - Labels, Selectors e Annotations]]

---

# O problema que este capítulo resolve

Imagine que você criou um Pod.

```text
Pod

↓

Running
```

Algum tempo depois...

```text
Pod

↓

Deleted
```

Quem percebe isso?

Quem cria outro?

Resposta:

**Ninguém.**

O Kubernetes não recria Pods "soltos".

É exatamente esse problema que o ReplicaSet resolve.

---

# O que é um ReplicaSet?

Um ReplicaSet é um **Controller** responsável por garantir que um determinado número de Pods esteja sempre em execução.

Ele não executa aplicações.

Ele monitora Pods.

Sua responsabilidade pode ser resumida em uma frase:

> "Garanta que existam exatamente X Pods correspondentes ao meu Selector."

---

# Modelo Mental

Imagine um gerente de uma equipe.

Você diz:

```text
Quero 5 funcionários trabalhando.
```

Durante o dia.

Um funcionário vai embora.

O gerente imediatamente contrata outro.

Você não precisa fazer nada.

ReplicaSet funciona exatamente assim.

---

# Relação entre ReplicaSet e Pods

Visualmente.

```text
ReplicaSet

↓

Selector

↓

Labels

↓

Pods
```

O ReplicaSet nunca conhece Pods diretamente.

Ele procura Pods através de Selectors.

---

# Exemplo

ReplicaSet.

```yaml
selector:

  matchLabels:

    app: backend
```

Pods.

```yaml
labels:

  app: backend
```

Resultado.

```text
ReplicaSet

↓

Pod A

Pod B

Pod C
```

---

# Como funciona o Reconciliation Loop?

Imagine:

```yaml
replicas: 3
```

Estado atual.

```text
Pod 1

Pod 2

Pod 3
```

Tudo certo.

Agora alguém executa.

```bash
kubectl delete pod pod-2
```

Situação.

```text
Pod 1

Pod 3
```

O ReplicaSet observa.

```text
Desejado

↓

3 Pods
```

```text
Atual

↓

2 Pods
```

Existe diferença.

```text
↓

Criar novo Pod
```

Resultado.

```text
Pod 1

Pod 3

Pod 4
```

Perceba.

O Pod antigo não voltou.

Foi criado um novo.

---

# Internals

O ReplicaSet executa continuamente o seguinte algoritmo.

```text
Consultar API Server

↓

Encontrar Pods

↓

Aplicar Selector

↓

Contar Pods

↓

Quantidade correta?

↓

Sim → Esperar

↓

Não

↓

Criar ou remover Pods
```

Esse ciclo acontece continuamente.

---

# Anatomia de um ReplicaSet

Exemplo.

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:

  name: backend

spec:

  replicas: 3

  selector:

    matchLabels:

      app: backend

  template:

    metadata:

      labels:

        app: backend

    spec:

      containers:

      - name: nginx

        image: nginx:latest
```

---

# Explicando o manifesto

## replicas

```yaml
replicas: 3
```

Quantidade desejada.

---

## selector

```yaml
selector:

  matchLabels:

    app: backend
```

Define quais Pods pertencem ao ReplicaSet.

---

## template

Aqui acontece algo importante.

```yaml
template:
```

O ReplicaSet precisa saber como criar novos Pods.

Para isso ele armazena um molde.

Esse molde é chamado de **Pod Template**.

Sempre que um novo Pod precisar ser criado.

O ReplicaSet utiliza esse template.

---

# Pod Template

Pense em um molde.

```text
Molde

↓

Criar Pod

↓

Criar Pod

↓

Criar Pod
```

Todos os Pods criados serão iguais.

---

# Fluxo completo

```text
ReplicaSet

↓

Consultar API

↓

Encontrar Pods

↓

Existem 3?

↓

Não

↓

Criar Pod

↓

Scheduler

↓

Worker Node

↓

Container Runtime

↓

Pod Running
```

---

# Escalando um ReplicaSet

Imagine.

```yaml
replicas: 3
```

Agora alteramos.

```yaml
replicas: 8
```

O ReplicaSet percebe.

```text
Desejado

↓

8
```

Atual.

```text
3
```

Resultado.

```text
Criar mais 5 Pods
```

---

# Escalando para baixo

Agora.

```yaml
replicas: 2
```

ReplicaSet.

```text
Desejado

↓

2
```

Atual.

```text
8
```

Resultado.

```text
Remover 6 Pods
```

---

# E se um Node morrer?

Imagine.

```text
Node A

Pod 1

Pod 2
```

O Node desliga.

```text
Node A

Offline
```

Os Pods desaparecem.

O ReplicaSet percebe.

```text
Faltam Pods.
```

Novos Pods serão criados.

Depois.

O Scheduler escolherá outros Nodes disponíveis.

Esse comportamento é um dos pilares da alta disponibilidade do Kubernetes.

---

# Comandos úteis

Criar.

```bash
kubectl apply -f replicaset.yaml
```

Listar.

```bash
kubectl get rs
```

Detalhes.

```bash
kubectl describe rs backend
```

Excluir.

```bash
kubectl delete rs backend
```

---

# Hands-on

Arquivo.

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:

  name: nginx-rs

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

        image: nginx:latest
```

Aplicar.

```bash
kubectl apply -f replicaset.yaml
```

Verificar.

```bash
kubectl get rs
```

```bash
kubectl get pods
```

Agora delete um Pod.

```bash
kubectl delete pod <nome-do-pod>
```

Observe.

```bash
kubectl get pods -w
```

Você verá imediatamente um novo Pod sendo criado.

Esse experimento demonstra o Reconciliation Loop em ação.

---

# Como isso aparece na prática

Embora possamos criar ReplicaSets manualmente, isso raramente acontece.

Na prática.

Quem cria ReplicaSets é o Deployment.

Fluxo real.

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Ou seja.

Você normalmente trabalha com Deployments.

E eles trabalham com ReplicaSets.

---

# Boas práticas

- Não utilize ReplicaSets diretamente em aplicações de produção.
- Utilize Deployments para aplicações stateless.
- Nunca altere Labels utilizadas pelo Selector.
- Utilize ReplicaSets apenas para aprendizado ou casos muito específicos.

---

# Erros comuns

> [!WARNING]

Excluir um Pod esperando que ele permaneça removido.

Se houver um ReplicaSet controlando esse Pod, outro será criado imediatamente.

---

> [!WARNING]

Modificar Labels dos Pods.

O ReplicaSet pode perder o controle desses Pods e criar novos.

---

> [!WARNING]

Criar Selectors muito genéricos.

Um ReplicaSet pode assumir Pods que não deveria controlar.

---

# Resumo

Neste capítulo aprendemos que:

- ReplicaSets são Controllers.
- Eles garantem a quantidade desejada de Pods.
- Utilizam Selectors para localizar Pods.
- Utilizam um Pod Template para criar novos Pods.
- Implementam o Reconciliation Loop.
- Normalmente são gerenciados por Deployments.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que é um ReplicaSet?
- [ ] Qual problema ele resolve?
- [ ] Como encontra seus Pods?
- [ ] O que é um Pod Template?
- [ ] O que acontece quando um Pod controlado é removido?
- [ ] Por que normalmente utilizamos Deployments em vez de ReplicaSets?

---

# Conectando os conceitos

Até este ponto, a arquitetura que construímos é:

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
                     │
             Container Runtime
                     │
                 Containers
```

Nos próximos capítulos adicionaremos novas camadas a esse diagrama.
