---
tags:
  - kubernetes
  - architecture
  - production
  - microservices
  - monolith
  - kafka
  - rabbitmq
  - multi-tenant
  - ha
  - platform-engineering
---
> [!INFO]
> Objetivo
>
> Compreender como arquitetar aplicações reais utilizando Kubernetes, avaliando padrões, trade-offs e decisões comuns em ambientes de produção.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Projetar arquiteturas para Kubernetes.
- [ ] Escolher entre monólitos e microsserviços.
- [ ] Definir estratégias para bancos de dados.
- [ ] Organizar ambientes e Namespaces.
- [ ] Projetar aplicações escaláveis.
- [ ] Entender arquiteturas enterprise.

---

# Introdução

Aprender Kubernetes é conhecer suas peças.

Projetar plataformas é saber como combiná-las.

Em produção, dificilmente teremos apenas um Deployment.

Uma aplicação costuma envolver:

- APIs
- Workers
- Filas
- Bancos
- Cache
- Observabilidade
- Segurança
- CI/CD
- Balanceamento
- Storage
- Ingress
- DNS

O arquiteto precisa pensar no sistema como um todo.

---

# Mental Model

Pense em Kubernetes como um sistema operacional para Data Centers.

Assim como um sistema operacional fornece:

- memória
- processos
- arquivos
- rede

O Kubernetes fornece:

- computação
- armazenamento
- rede
- escalabilidade
- disponibilidade

Sua aplicação apenas utiliza esses recursos.

---

# Arquitetura 1 — Monólito Modernizado

Muitas empresas ainda possuem aplicações monolíticas.

Isso não significa que elas não possam utilizar Kubernetes.

Exemplo:

```text
                Internet
                     │
             Load Balancer
                     │
                 Ingress
                     │
                 Service
                     │
             Deployment
                     │
                Monólito
                     │
                PostgreSQL
```

---

## Quando utilizar

- aplicações pequenas;
- equipes pequenas;
- MVPs;
- sistemas internos;
- baixa complexidade.

---

## Vantagens

- simplicidade;
- deploy único;
- menos comunicação entre serviços;
- debugging mais fácil.

---

## Desvantagens

- escalabilidade limitada;
- deploy de tudo ao mesmo tempo;
- maior acoplamento.

---

# Arquitetura 2 — Microsserviços

```text
             Internet
                  │
             API Gateway
                  │
      ┌───────────┼────────────┐
      │           │            │
    Auth        Orders      Payments
      │           │            │
      └───────────┼────────────┘
                  │
             RabbitMQ/Kafka
                  │
          Workers Assíncronos
```

---

Cada serviço possui:

- Deployment
- Service
- ConfigMap
- Secret
- HPA

Independência é a principal vantagem.

---

## Quando utilizar

- equipes grandes;
- produtos complexos;
- escalabilidade independente;
- deploy frequente.

---

## Trade-offs

Vantagens:

- deploy independente;
- escalabilidade isolada;
- autonomia das equipes.

Desvantagens:

- maior complexidade;
- observabilidade obrigatória;
- comunicação distribuída;
- tracing necessário.

---

# Arquitetura 3 — Event-Driven

Muito utilizada em sistemas modernos.

```text
API

↓

Kafka

↓

Consumers

↓

Banco
```

ou

```text
API

↓

RabbitMQ

↓

Workers

↓

Serviços
```

---

## Casos de uso

- processamento de pagamentos;
- envio de e-mails;
- geração de PDFs;
- processamento de imagens;
- integrações.

---

# Workers

Um Worker normalmente possui apenas:

```text
Deployment

↓

Fila

↓

Processamento
```

Não existe Ingress.

Não existe acesso externo.

---

# APIs REST + Workers

Arquitetura extremamente comum.

```text
Cliente

↓

API

↓

Banco

↓

RabbitMQ

↓

Worker
```

A API responde rapidamente.

O processamento ocorre em background.

---

# GraphQL Gateway

Outra arquitetura comum.

```text
Cliente

↓

GraphQL

↓

Microsserviços
```

O Gateway agrega múltiplos serviços.

---

# Cache

Quase toda aplicação utiliza cache.

Normalmente.

```text
API

↓

Redis

↓

Banco
```

---

# Banco dentro ou fora do Cluster?

Essa é uma das dúvidas mais comuns.

---

## Banco dentro

```text
StatefulSet

↓

PVC

↓

Disco
```

---

### Vantagens

- laboratório;
- desenvolvimento;
- ambientes pequenos.

---

### Desvantagens

- backup mais complexo;
- alta disponibilidade mais difícil;
- operação maior.

---

## Banco Gerenciado

```text
API

↓

RDS

ou

Cloud SQL

ou

Azure Database
```

---

### Vantagens

- backup automático;
- alta disponibilidade;
- manutenção simplificada;
- upgrades gerenciados.

---

### Minha recomendação

Produção:

Banco gerenciado.

Laboratório:

StatefulSet.

---

# Organização de Namespaces

Exemplo.

```text
production

staging

development

monitoring

logging

ingress

cert-manager

argocd
```

Evite colocar tudo em `default`.

---

# Cluster único ou múltiplos?

## Cluster único

```text
Cluster

↓

Namespaces
```

Mais simples.

---

## Multi-cluster

```text
Produção

Homologação

Desenvolvimento
```

Maior isolamento.

Maior custo.

---

# Multi-Tenant

Existem três estratégias.

---

## Namespace por cliente

```text
cliente-a

cliente-b

cliente-c
```

Boa para poucos clientes.

---

## Cluster por cliente

Muito isolamento.

Maior custo.

---

## Tenant lógico

Todos compartilham recursos.

A aplicação faz o isolamento.

É o modelo mais comum para SaaS.

---

# Organização de Projetos

Uma estrutura bastante utilizada.

```text
infra/

apps/

platform/

monitoring/

gitops/
```

Separação clara entre responsabilidades.

---

# Arquitetura de Plataforma

```text
                    Backstage
                         │
                    GitHub
                         │
                  GitHub Actions
                         │
                     ArgoCD
                         │
                  Kubernetes
                         │
      ┌──────────┬──────────┬──────────┐
      │          │          │
   Prometheus  Grafana     Loki
      │          │          │
      └──────────┴──────────┘
```

---

# Alta Disponibilidade

Boas práticas.

- múltiplas zonas;
- múltiplas réplicas;
- readiness probe;
- PodDisruptionBudget;
- HPA;
- Cluster Autoscaler.

---

# Escalabilidade

Escalas diferentes.

Aplicação.

↓

HPA

Infraestrutura.

↓

Cluster Autoscaler

Banco.

↓

Read Replicas

---

# Segurança

Nunca exponha Pods.

Fluxo.

```text
Internet

↓

Load Balancer

↓

Ingress

↓

Service

↓

Pods
```

---

# Observabilidade

Toda arquitetura deve possuir.

- métricas;
- logs;
- tracing;
- alertas.

Sem observabilidade.

Não existe produção.

---

# Arquitetura Enterprise

```text
Usuário
     │
 CDN/WAF
     │
 Load Balancer
     │
 API Gateway
     │
 Ingress
     │
 Kubernetes
     │
 ┌──────────────┬──────────────┐
 │              │              │
 APIs        Workers      CronJobs
 │              │              │
 └──────┬───────┴──────────────┘
        │
 Redis / Kafka
        │
 Banco Gerenciado
        │
 Observabilidade
```

---

# Quando NÃO usar Kubernetes?

Nem todo sistema precisa de Kubernetes.

Considere alternativas quando:

- uma única aplicação pequena;
- baixo volume de usuários;
- equipe reduzida;
- custo operacional elevado;
- deploys pouco frequentes.

Nesses casos, uma VM ou um serviço como AWS ECS/Fargate, Azure Container Apps ou Azure Web Apps pode ser mais adequado.

---

# Arquitetura recomendada para um SaaS moderno

```text
GitHub
    │
GitHub Actions
    │
Container Registry
    │
ArgoCD
    │
Kubernetes
    │
──────────────
Ingress
──────────────
API
Workers
CronJobs
──────────────
Redis
Kafka
──────────────
RDS
──────────────
Prometheus
Grafana
Loki
Tempo
```

---

# Boas práticas

- Prefira bancos gerenciados em produção.
- Utilize Namespaces para organizar ambientes.
- Automatize toda a plataforma.
- Separe infraestrutura da aplicação.
- Utilize GitOps.
- Não transforme microsserviços em objetivo; eles são uma solução para problemas específicos.
- Mantenha a arquitetura o mais simples possível.

---

# Erros comuns

> [!WARNING]

Migrar para microsserviços sem necessidade.

---

> [!WARNING]

Executar todos os componentes no Namespace `default`.

---

> [!WARNING]

Executar bancos críticos sem estratégia de backup e recuperação.

---

> [!WARNING]

Ignorar observabilidade desde o início do projeto.

---

# Resumo

Neste capítulo aprendemos que:

- Kubernetes suporta diferentes estilos arquiteturais.
- Monólitos continuam sendo válidos em muitos cenários.
- Microsserviços oferecem flexibilidade ao custo de maior complexidade.
- Event-Driven é uma excelente abordagem para processamento assíncrono.
- Bancos gerenciados são, em geral, a melhor escolha para produção.
- A organização da plataforma (Namespaces, clusters, GitOps e observabilidade) é tão importante quanto a aplicação.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Quando utilizar um monólito em Kubernetes?
- [ ] Quais os principais trade-offs dos microsserviços?
- [ ] Quando utilizar Kafka ou RabbitMQ?
- [ ] Banco dentro ou fora do cluster?
- [ ] Como organizar Namespaces em produção?
- [ ] Quando utilizar múltiplos clusters?
- [ ] Como implementar multi-tenancy?
- [ ] Quais componentes compõem uma plataforma Kubernetes moderna?
- [ ] Em quais cenários Kubernetes pode não ser a melhor escolha?

---

# Diagrama consolidado

```text
                    GitHub
                       │
                GitHub Actions
                       │
             Container Registry
                       │
                    ArgoCD
                       │
                 Kubernetes
                       │
      ┌────────────────┼────────────────┐
      │                │                │
    Ingress         Services        CronJobs
      │                │                │
      ├───────┬────────┴─────────┐
      │       │                  │
     APIs   Workers         Consumers
      │       │                  │
      └───────┼──────────────────┘
              │
         Redis / Kafka
              │
        Banco Gerenciado
              │
    Prometheus / Grafana / Loki
```
