---
tags:
  - kubernetes
  - cicd
  - github-actions
  - gitlab-ci
  - jenkins
  - argocd
  - helm
  - gitops
  - devops
---
> [!INFO]
> Objetivo
>
> Entender como construir pipelines modernos para aplicações Kubernetes utilizando integração contínua (CI), entrega contínua (CD), GitOps e automação de deploy.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Diferenciar CI e CD.
- [ ] Compreender um pipeline moderno.
- [ ] Integrar Docker com Kubernetes.
- [ ] Publicar imagens em Registries.
- [ ] Automatizar Deploys.
- [ ] Utilizar GitOps com ArgoCD.
- [ ] Escolher ferramentas de CI/CD.

---

# Pré-requisitos

- Docker
- Git
- [[18 - Kubernetes em Produção]]
- [[19 - Ecossistema Cloud Native]]
- [[20 - Kubernetes na Cloud]]

---

# Modelo Mental

Imagine uma fábrica.

O desenvolvedor produz uma peça.

Essa peça precisa passar por várias etapas de qualidade antes de chegar ao cliente.

O pipeline faz exatamente isso.

Cada etapa valida algo.

```text
Código

↓

Testes

↓

Imagem Docker

↓

Segurança

↓

Deploy

↓

Produção
```

---

# O que é CI?

Continuous Integration.

Objetivo.

Integrar código continuamente.

Sempre que ocorre um.

```text
git push
```

Executamos automaticamente.

- Build
- Testes
- Lint
- Static Analysis
- Segurança

Se qualquer etapa falhar.

Nada continua.

---

# O que é CD?

Existem duas interpretações.

## Continuous Delivery

A aplicação fica pronta para produção.

Mas alguém aprova.

---

## Continuous Deployment

Após aprovação automática.

A aplicação é publicada imediatamente.

Sem intervenção humana.

---

# Fluxo completo

```text
Developer

↓

Git Push

↓

CI

↓

Docker Build

↓

Registry

↓

Atualiza GitOps

↓

ArgoCD

↓

Cluster Kubernetes

↓

Produção
```

---

# Pipeline moderno

Um pipeline profissional normalmente possui.

```text
Checkout

↓

Lint

↓

Testes

↓

Build

↓

Docker Build

↓

Docker Scan

↓

Push Registry

↓

Atualização GitOps

↓

Deploy

↓

Smoke Tests

↓

Notificação
```

---

# Ferramentas de CI

As mais comuns atualmente.

- GitHub Actions
- GitLab CI
- Jenkins
- Azure DevOps
- CircleCI
- Bitbucket Pipelines

---

# GitHub Actions

Hoje é uma das ferramentas mais utilizadas.

Cada pipeline é definido em YAML.

Exemplo.

```yaml
name: CI

on:

  push:

jobs:

  build:
```

---

# Conceitos importantes

Workflow

↓

Jobs

↓

Steps

Cada Job executa em um Runner.

---

# GitLab CI

Muito parecido.

Arquivo.

```text
.gitlab-ci.yml
```

Conceitos.

- stages
- jobs
- runners

---

# Jenkins

Durante muitos anos foi o padrão.

Hoje continua muito utilizado.

Principal vantagem.

Alta flexibilidade.

Principal desvantagem.

Maior custo operacional.

---

# Container Registry

Após o Build.

Precisamos armazenar a imagem.

Exemplos.

- Docker Hub
- GitHub Container Registry
- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Harbor

---

# Versionamento de imagens

Evite.

```text
latest
```

Prefira.

```text
v1.2.5

ou

Git SHA
```

Exemplo.

```text
backend

↓

backend:7c18f42
```

---

# Segurança

Antes do Push.

Escaneie a imagem.

Ferramentas.

- Trivy
- Grype
- Snyk

Nunca publique imagens vulneráveis.

---

# Atualização do Deployment

Existem duas abordagens.

---

## Imperativa

```bash
kubectl set image
```

Pouco utilizada em produção.

---

## Declarativa

Atualizar.

```yaml
image:

  tag: 1.8.3
```

Depois.

GitOps sincroniza.

---

# GitOps

Já estudamos.

Agora dentro do pipeline.

Fluxo.

```text
Pipeline

↓

Atualiza Helm

↓

Commit

↓

Git

↓

ArgoCD

↓

Deploy
```

Observe.

O pipeline não conversa com o cluster.

Quem conversa é o ArgoCD.

---

# Helm no pipeline

Imagine.

Alterar apenas.

```yaml
image:

  tag: 2.1.0
```

O restante permanece igual.

Muito mais simples.

---

# Kustomize

Outra possibilidade.

Aplicar patches.

Exemplo.

Produção.

```text
Replicas = 10
```

Homologação.

```text
Replicas = 2
```

---

# Estratégias de Deploy

O pipeline pode utilizar.

- Rolling Update
- Blue/Green
- Canary

Cada uma atende objetivos diferentes.

---

# Rollback

Caso algo falhe.

Pipeline.

↓

Helm Rollback.

↓

ArgoCD sincroniza.

↓

Cluster retorna para versão anterior.

---

# Testes

Pipeline moderno normalmente executa.

- Unitários
- Integração
- Contrato
- Segurança
- Smoke Tests

---

# Smoke Test

Após o Deploy.

Executar.

```text
GET /health
```

Se falhar.

Rollback.

---

# Ambientes

Normalmente.

```text
Development

↓

Homologação

↓

Produção
```

Cada ambiente possui seu próprio Namespace ou Cluster.

---

# Secrets

Nunca armazene.

```yaml
PASSWORD=123
```

No pipeline.

Utilize.

- GitHub Secrets
- GitLab Variables
- Azure Key Vault
- AWS Secrets Manager
- HashiCorp Vault

---

# Fluxo GitHub Actions

```text
Git Push

↓

Runner

↓

Docker Build

↓

Docker Push

↓

Atualiza Helm

↓

Git Push

↓

ArgoCD

↓

Cluster
```

---

# Fluxo GitLab CI

```text
Git Push

↓

Pipeline

↓

Build

↓

Registry

↓

GitOps

↓

Deploy
```

---

# Ferramentas complementares

| Categoria | Ferramenta |
|------------|------------|
| CI | GitHub Actions |
| CI | GitLab CI |
| CI | Jenkins |
| Registry | ECR |
| Registry | ACR |
| Registry | GHCR |
| CD | ArgoCD |
| Templates | Helm |
| Customização | Kustomize |

---

# Hands-on

Executar pipeline localmente.

```bash
docker build .
```

---

Publicar imagem.

```bash
docker push
```

---

Atualizar tag.

```yaml
image:

  tag: 1.5.0
```

---

Ver sincronização.

```bash
argocd app get api
```

---

Sincronizar manualmente.

```bash
argocd app sync api
```

---

# Arquitetura completa

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

Docker Build

↓

Container Registry

↓

GitOps Repository

↓

ArgoCD

↓

Kubernetes

↓

Observabilidade
```

---

# Pipeline Enterprise

```text
Checkout

↓

Lint

↓

Unit Tests

↓

Build

↓

Docker Build

↓

Image Scan

↓

Push Registry

↓

Atualiza Helm

↓

Git Commit

↓

ArgoCD

↓

Deploy

↓

Smoke Tests

↓

Alertas
```

---

# Boas práticas

- Automatize tudo que for repetitivo.
- Utilize GitOps para Deploy.
- Nunca faça deploy manual em produção.
- Versione imagens de forma imutável.
- Execute testes antes do Build.
- Escaneie imagens antes da publicação.
- Utilize ambientes separados.
- Automatize rollback quando possível.

---

# Erros comuns

> [!WARNING]

Utilizar a tag `latest` para imagens.

---

> [!WARNING]

Executar `kubectl apply` diretamente no pipeline.

Prefira GitOps.

---

> [!WARNING]

Misturar código da aplicação e manifestos GitOps no mesmo repositório sem uma estratégia clara.

---

> [!WARNING]

Publicar imagens sem testes ou análise de vulnerabilidades.

---

# Resumo

Neste capítulo aprendemos que:

- CI integra código continuamente.
- CD automatiza a entrega e o deploy.
- Pipelines modernos incluem testes, build, segurança e publicação.
- Registries armazenam imagens Docker.
- GitOps desacopla o pipeline do cluster.
- Helm e Kustomize simplificam a gestão dos manifestos.
- Rollback deve fazer parte da estratégia de entrega.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual a diferença entre Continuous Delivery e Continuous Deployment?
- [ ] Quais etapas compõem um pipeline moderno?
- [ ] Por que evitar a tag `latest`?
- [ ] Qual o papel de um Container Registry?
- [ ] Como GitOps altera a forma de fazer deploy?
- [ ] Quando utilizar Helm ou Kustomize?
- [ ] Como implementar rollback automático?
- [ ] Onde armazenar Secrets utilizados pelo pipeline?
- [ ] Como integrar GitHub Actions com ArgoCD?

---

# Diagrama consolidado

```text
                  Developer
                      │
                  Git Push
                      │
             GitHub / GitLab
                      │
                 CI Pipeline
                      │
      ┌───────────────┼────────────────┐
      │               │                │
   Lint/Testes   Docker Build    Image Scan
      │               │                │
      └───────────────┼────────────────┘
                      ▼
             Container Registry
                      │
            Atualização GitOps
                      │
                  ArgoCD
                      │
              Kubernetes Cluster
                      │
         Rolling / Canary / Blue-Green
                      │
             Observabilidade
                      │
                  Usuários
```
