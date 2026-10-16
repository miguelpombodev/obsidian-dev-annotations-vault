---
tags:
  - kubernetes
  - labels
  - selectors
  - annotations
---
> [!INFO] Objetivo
> Entender como o Kubernetes organiza, identifica e relaciona recursos através de Labels, Selectors e Annotations.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender o propósito das Labels.
- [ ] Explicar como funcionam os Selectors.
- [ ] Diferenciar Labels de Annotations.
- [ ] Criar Labels seguindo boas práticas.
- [ ] Utilizar Selectors para consultar recursos.
- [ ] Entender como praticamente todos os recursos do Kubernetes utilizam Labels.

---

# Pré-requisitos

- [[01 - Por que Kubernetes existe]]
- [[02 - Arquitetura do Kubernetes]]
- [[03 - Objetos da API]]
- [[04 - Pods]]

---

# O problema que este capítulo resolve

Imagine um cluster com centenas de Pods.

```text
Pod A

Pod B

Pod C

Pod D

Pod E

Pod F

...
```

Como um Service sabe quais Pods pertencem a ele?

Como um Deployment identifica seus Pods?

Como um ReplicaSet encontra os Pods que precisa manter?

A resposta é:

**Labels.**

---

# O que são Labels?

Labels são pares **chave/valor** adicionados aos objetos do Kubernetes.

Exemplo:

```yaml
labels:
  app: ecommerce
```

Outro exemplo:

```yaml
labels:
  app: api
  environment: production
  team: backend
```

Uma Label não muda o comportamento do recurso.

Ela apenas adiciona uma informação estruturada.

Pense nelas como **tags**.

---

# Modelo Mental

Imagine uma caixa de arquivos.

Cada documento recebe etiquetas.

```text
📄 Documento

🏷 Financeiro
🏷 2025
🏷 Pago
```

Agora imagine milhares de documentos.

Encontrar todos os documentos pagos fica trivial.

O Kubernetes faz exatamente isso.

---

# Onde podemos utilizar Labels?

Praticamente em qualquer objeto.

Por exemplo:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Namespaces
- Nodes
- PersistentVolumes

---

# Anatomia de uma Label

Uma Label possui:

```text
chave

↓

valor
```

Exemplo.

```yaml
labels:
  app: backend
```

Outra.

```yaml
labels:
  environment: production
```

Outra.

```yaml
labels:
  version: v2
```

---

# Um objeto pode possuir várias Labels

Exemplo.

```yaml
metadata:

  labels:

    app: pedidos

    environment: production

    version: v2

    team: backend

    owner: squad-a
```

Isso permite múltiplas formas de organização.

---

# Convenções de nomenclatura

Boas Labels são simples e previsíveis.

Exemplo.

```yaml
labels:

  app: payments

  tier: backend

  environment: production

  version: v1
```

Evite:

```yaml
labels:

  servidor-da-api-do-joao: verdadeiro
```

Labels devem representar características do recurso.

Não detalhes temporários.

---

# O que são Selectors?

Se Labels identificam objetos...

Selectors encontram esses objetos.

Exemplo.

Um Service deseja localizar Pods.

```text
Service

↓

Selector

↓

app=backend

↓

Pod A ✅

Pod B ✅

Pod C ❌
```

O Selector consulta Labels.

Sempre.

---

# Match Labels

O tipo mais simples.

```yaml
selector:

  matchLabels:

    app: backend
```

Significa.

```text
Quero todos os Pods

onde

app = backend
```

---

# Exemplo completo

Pod.

```yaml
metadata:

  labels:

    app: backend
```

Service.

```yaml
selector:

  app: backend
```

Resultado.

```text
Service

↓

Pod
```

Sem IP.

Sem hostname.

Sem configuração manual.

Apenas Labels.

---

# Internals

O Kubernetes não cria ligações físicas entre recursos.

Não existe algo como:

```text
Deployment

↓

Pod 1
```

O que existe é:

```text
Deployment

↓

Selector

↓

Consulta API

↓

Labels

↓

Pods encontrados
```

Os relacionamentos são sempre resolvidos dinamicamente.

Isso é extremamente importante.

---

# Como um Deployment encontra seus Pods?

Imagine:

```yaml
labels:

  app: ecommerce
```

Deployment.

```yaml
selector:

  matchLabels:

    app: ecommerce
```

O Deployment consulta constantemente a API.

```text
Existe algum Pod

com

app=ecommerce ?
```

Se existir.

Ótimo.

Caso contrário.

Criar novo Pod.

---

# O mesmo acontece com Services

Imagine três Pods.

```text
Pod A

app=api
```

```text
Pod B

app=api
```

```text
Pod C

app=worker
```

O Service possui:

```yaml
selector:

  app: api
```

Resultado.

```text
Service

↓

Pod A

↓

Pod B
```

O Pod C é ignorado.

---

# Match Expressions

Além de igualdade.

Também podemos utilizar expressões.

```yaml
selector:

  matchExpressions:

    - key: environment

      operator: In

      values:

        - production

        - staging
```

Outros operadores.

- In
- NotIn
- Exists
- DoesNotExist

São utilizados principalmente em afinidade e políticas.

---

# Consultando Labels

Podemos listar Pods.

```bash
kubectl get pods
```

Mostrar Labels.

```bash
kubectl get pods --show-labels
```

Filtrar.

```bash
kubectl get pods -l app=backend
```

Outro exemplo.

```bash
kubectl get pods -l environment=production
```

Múltiplos filtros.

```bash
kubectl get pods -l app=backend,environment=production
```

---

# Alterando Labels

Adicionar.

```bash
kubectl label pod nginx version=v2
```

Remover.

```bash
kubectl label pod nginx version-
```

---

# O que são Annotations?

Annotations parecem Labels.

Mas possuem objetivo diferente.

Exemplo.

```yaml
annotations:

  owner: squad-backend

  documentation: wiki.company/api

  commit: 94f1d2c
```

As diferenças.

Labels.

- utilizadas para seleção;
- utilizadas por Controllers;
- utilizadas por Services.

Annotations.

- apenas armazenam informações.

O Kubernetes nunca utiliza uma Annotation para localizar recursos.

---

# Labels x Annotations

| Labels | Annotations |
|---------|-------------|
| Seleção | Documentação |
| Indexadas | Não indexadas |
| Pequenas | Podem ser grandes |
| Utilizadas pelos Controllers | Apenas metadados |

---

# Hands-on

Criar Pod.

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

  labels:

    app: nginx

    environment: dev

spec:

  containers:

    - name: nginx

      image: nginx
```

Aplicar.

```bash
kubectl apply -f pod.yaml
```

Consultar.

```bash
kubectl get pods --show-labels
```

Filtrar.

```bash
kubectl get pods -l app=nginx
```

---

# Como isso aparece na prática

Imagine um Deployment.

Ele cria Pods.

Esses Pods recebem.

```yaml
labels:

  app: api
```

Um Service.

```yaml
selector:

  app: api
```

Automaticamente.

Sempre que novos Pods surgirem.

O Service começa a enviar tráfego para eles.

Sem qualquer configuração adicional.

Esse desacoplamento é um dos pilares da arquitetura do Kubernetes.

---

# Boas práticas

- Utilize nomes consistentes.
- Não utilize Labels para armazenar textos grandes.
- Utilize Labels para classificação.
- Utilize Annotations para documentação.
- Defina convenções para toda a equipe.

Exemplo recomendado.

```yaml
labels:

  app:

  component:

  version:

  environment:

  team:
```

---

# Erros comuns

> [!WARNING]

Criar Services utilizando nomes de Pods.

Sempre utilize Labels.

---

> [!WARNING]

Utilizar Annotations para seleção.

Selectors ignoram Annotations.

---

> [!WARNING]

Alterar Labels utilizadas por um Deployment.

Isso pode fazer com que o Deployment "perca" seus Pods e crie novos.

---

# Resumo

Neste capítulo aprendemos que:

- Labels identificam recursos.
- Selectors localizam recursos.
- Deployments utilizam Selectors para encontrar Pods.
- Services utilizam Selectors para enviar tráfego.
- ReplicaSets também utilizam Selectors.
- Annotations armazenam informações adicionais.
- A maioria dos relacionamentos do Kubernetes é baseada em Labels.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] O que são Labels?
- [ ] Qual a diferença entre Labels e Annotations?
- [ ] Como um Service encontra um Pod?
- [ ] Como um Deployment encontra seus Pods?
- [ ] O que faz um Selector?
- [ ] Quando utilizar Match Expressions?
