---
tags:
  - kubernetes
  - statefulset
  - storage
  - database
  - persistent-storage
---
> [!INFO] Objetivo
> Entender por que aplicações stateful exigem um controlador diferente dos Deployments, compreender como o StatefulSet fornece identidade estável, armazenamento persistente e inicialização ordenada.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar por que bancos de dados normalmente utilizam StatefulSets.
- [ ] Entender a diferença entre Deployment e StatefulSet.
- [ ] Compreender o conceito de identidade estável.
- [ ] Entender como funciona o volumeClaimTemplates.
- [ ] Explicar a criação e remoção ordenada dos Pods.
- [ ] Escolher corretamente entre Deployment e StatefulSet.

---

# Pré-requisitos

- [[07 - Deployments]]
- [[08 - Services]]
- [[10 - Volumes, PersistentVolumes e PersistentVolumeClaims]]

---

# O problema que este capítulo resolve

Imagine um banco PostgreSQL executando em um Deployment.

```text
Deployment

↓

Pod postgres

↓

PVC
```

Tudo funciona.

Até que o Pod morre.

O Deployment cria outro.

```text
postgres-x82df
```

Depois.

```text
postgres-q91ad
```

Depois.

```text
postgres-k2d9s
```

O banco continua funcionando.

Mas...

E se existirem três réplicas?

Como uma réplica encontra a outra?

Como um cluster PostgreSQL identifica o nó líder?

Como um cluster Kafka sabe quem é quem?

É aqui que surge o problema.

---

# O grande problema

Deployments tratam Pods como descartáveis.

```text
Pod A

↓

Morre

↓

Novo Pod
```

Para uma API REST isso é perfeito.

Para um banco distribuído isso é um desastre.

Porque cada instância precisa possuir uma identidade permanente.

---

# Modelo Mental

Imagine um hotel.

Deployment.

```text
Quarto disponível.

Você ocupa qualquer um.
```

Hoje é o quarto 12.

Amanhã o 27.

Depois o 31.

Não importa.

Agora imagine um cartório.

Cada funcionário possui uma mesa fixa.

Sempre.

Não importa o dia.

Essa mesa representa a identidade do StatefulSet.

---

# O que é um StatefulSet?

Um StatefulSet é um Controller responsável por aplicações que precisam de:

- identidade estável;
- armazenamento persistente;
- inicialização ordenada;
- desligamento ordenado.

---

# Quando utilizar?

Normalmente.

- PostgreSQL
- MySQL
- MongoDB
- Cassandra
- Kafka
- RabbitMQ
- Elasticsearch
- Redis Cluster
- Zookeeper
- etcd

---

# Quando NÃO utilizar?

- APIs REST
- Frontend React
- FastAPI
- Django
- Flask
- Nginx
- Workers stateless

Esses continuam utilizando Deployments.

---

# Arquitetura

Deployment.

```text
Deployment

↓

ReplicaSet

↓

Pods
```

StatefulSet.

```text
StatefulSet

↓

Pods
```

Observe.

O StatefulSet não utiliza ReplicaSets.

Ele gerencia diretamente seus Pods.

---

# Identidade estável

Esse é o conceito mais importante.

Imagine três Pods.

```text
postgres-0

postgres-1

postgres-2
```

Esses nomes nunca mudam.

Mesmo que o Pod seja recriado.

```text
postgres-0

↓

Delete

↓

postgres-0
```

Não surge.

```text
postgres-x91ad
```

A identidade permanece.

---

# DNS estável

Como consequência.

Cada Pod recebe um DNS fixo.

```text
postgres-0.database.default.svc.cluster.local
```

```text
postgres-1.database.default.svc.cluster.local
```

```text
postgres-2.database.default.svc.cluster.local
```

Isso permite que aplicações distribuídas encontrem seus pares.

---

# Ordem de criação

Deployment.

```text
Criar tudo ao mesmo tempo.
```

StatefulSet.

```text
postgres-0

↓

postgres-1

↓

postgres-2
```

Somente após o Pod anterior estar saudável.

---

# Ordem de remoção

Também acontece em ordem inversa.

```text
postgres-2

↓

postgres-1

↓

postgres-0
```

Esse comportamento protege clusters distribuídos.

---

# volumeClaimTemplates

Outra funcionalidade extremamente importante.

Imagine.

Três Pods.

Cada um precisa do próprio disco.

O StatefulSet cria automaticamente.

```text
postgres-0

↓

PVC

↓

PV
```

Depois.

```text
postgres-1

↓

PVC

↓

PV
```

Depois.

```text
postgres-2

↓

PVC

↓

PV
```

Cada Pod recebe seu próprio armazenamento.

---

# Anatomia de um StatefulSet

```yaml
apiVersion: apps/v1

kind: StatefulSet

metadata:

  name: postgres

spec:

  serviceName: postgres

  replicas: 3

  selector:

    matchLabels:

      app: postgres

  template:

    metadata:

      labels:

        app: postgres

    spec:

      containers:

      - name: postgres

        image: postgres:16

        volumeMounts:

        - name: dados

          mountPath: /var/lib/postgresql/data

  volumeClaimTemplates:

  - metadata:

      name: dados

    spec:

      accessModes:

      - ReadWriteOnce

      resources:

        requests:

          storage: 20Gi
```

---

# O papel do serviceName

Todo StatefulSet exige um Service.

Mas não qualquer Service.

Normalmente utilizamos um **Headless Service**.

```yaml
clusterIP: None
```

Esse Service não faz balanceamento.

Ele apenas cria registros DNS.

Fluxo.

```text
Headless Service

↓

DNS

↓

postgres-0

postgres-1

postgres-2
```

---

# Headless Service

Enquanto um Service comum responde.

```text
postgres

↓

10.96.0.15
```

Um Headless Service responde.

```text
postgres-0
```

```text
postgres-1
```

```text
postgres-2
```

Cada Pod possui sua própria resolução DNS.

---

# Escalando

Imagine.

```yaml
replicas: 3
```

Depois.

```yaml
replicas: 5
```

Novos Pods.

```text
postgres-3

postgres-4
```

Observe.

A numeração continua.

---

# Recriando Pods

Imagine.

```bash
kubectl delete pod postgres-1
```

Novo Pod.

```text
postgres-1
```

Mesmo nome.

Mesmo PVC.

Mesmo DNS.

Mesmo disco.

---

# Fluxo completo

```text
StatefulSet

↓

Criar postgres-0

↓

Criar PVC

↓

Provisionar Disco

↓

Pod Running

↓

Criar postgres-1

↓

Criar PVC

↓

Provisionar Disco

↓

Pod Running

↓

Criar postgres-2
```

Tudo ocorre sequencialmente.

---

# Hands-on

Criar StatefulSet.

```bash
kubectl apply -f statefulset.yaml
```

Verificar.

```bash
kubectl get statefulsets
```

Listar Pods.

```bash
kubectl get pods
```

Resultado.

```text
postgres-0

postgres-1

postgres-2
```

Listar PVCs.

```bash
kubectl get pvc
```

Observe.

Cada Pod possui um PVC diferente.

---

# Deployment x StatefulSet

| Deployment | StatefulSet |
|------------|-------------|
| Stateless | Stateful |
| Pods descartáveis | Pods com identidade |
| ReplicaSet | Gerencia Pods diretamente |
| Nome aleatório | Nome previsível |
| Compartilha PVC (quando configurado) | Um PVC por Pod |
| Inicialização paralela | Inicialização ordenada |
| Desligamento paralelo | Desligamento ordenado |

---

# Como isso aparece na prática

Imagine um cluster PostgreSQL.

```text
Cliente

↓

Service

↓

postgres-0 (Primary)

↓

Replicação

↓

postgres-1

↓

postgres-2
```

Cada instância precisa manter sua identidade e seu armazenamento.

Se o `postgres-1` for recriado.

Ele continua sendo.

```text
postgres-1
```

Nunca outro nome.

---

# Relação com os capítulos anteriores

Nosso modelo mental evolui.

```text
Cliente
   │
Service
   │
Deployment ───────────────┐
   │                      │
ReplicaSet               StatefulSet
   │                      │
Pods                 Pods (Identidade)
   │                      │
Volumes           volumeClaimTemplates
   │                      │
PVC                  PVC por Pod
   │                      │
PV                      PV
```

---

# Boas práticas

- Utilize StatefulSets apenas quando necessário.
- Sempre utilize um Headless Service junto ao StatefulSet.
- Utilize `volumeClaimTemplates` em vez de compartilhar um único PVC.
- Não utilize StatefulSets apenas porque a aplicação possui banco de dados; muitas vezes o banco é um serviço gerenciado (RDS, Cloud SQL, Azure Database).

---

# Erros comuns

> [!WARNING]

Utilizar StatefulSet para APIs stateless.

Isso aumenta a complexidade sem oferecer benefícios.

---

> [!WARNING]

Compartilhar um único volume entre várias instâncias de banco sem suporte ao modo de acesso adequado.

Isso pode causar corrupção de dados.

---

> [!WARNING]

Achar que StatefulSet substitui um mecanismo de replicação do banco.

O Kubernetes garante identidade e armazenamento, mas não implementa replicação ou consenso. Esses mecanismos pertencem à própria aplicação (PostgreSQL, Kafka, Cassandra, etc.).

---

# Resumo

Neste capítulo aprendemos que:

- StatefulSets foram criados para aplicações stateful.
- Eles fornecem identidade estável para os Pods.
- Cada Pod recebe seu próprio PVC automaticamente.
- A criação e remoção dos Pods ocorre de forma ordenada.
- StatefulSets normalmente trabalham em conjunto com Headless Services.
- Deployments continuam sendo a melhor escolha para aplicações stateless.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que um banco de dados normalmente utiliza StatefulSet?
- [ ] Qual a diferença entre Deployment e StatefulSet?
- [ ] O que é identidade estável?
- [ ] Qual a função do `volumeClaimTemplates`?
- [ ] O que é um Headless Service?
- [ ] Em quais cenários você **não** utilizaria um StatefulSet?

---

# Diagrama consolidado

```text
                           Cliente
                              │
                           CoreDNS
                              │
                    Service / Headless Service
                       │               │
                 Deployment      StatefulSet
                       │               │
                  ReplicaSet     Pods (Identidade)
                       │               │
                      Pods       PVC por Pod
                       │               │
          ConfigMap  Secret      VolumeClaimTemplate
                 │      │               │
                 └──────┼───────────────┘
                        ▼
                    Containers
                        │
                       PVC
                        │
                        PV
                        │
                  StorageClass
                        │
                    CSI Driver
                        │
                  Cloud Storage
```
