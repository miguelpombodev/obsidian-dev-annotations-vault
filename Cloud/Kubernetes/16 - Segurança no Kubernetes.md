---
tags:
  - kubernetes
  - security
  - rbac
  - serviceaccount
  - pod-security
  - networkpolicy
  - cks
  - zero-trust
---
> [!INFO]
> Objetivo
>
> Compreender as principais camadas de segurança do Kubernetes, desde autenticação e autorização até proteção de Pods, imagens, rede e supply chain.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar como funciona a segurança no Kubernetes.
- [ ] Diferenciar autenticação e autorização.
- [ ] Entender RBAC.
- [ ] Compreender ServiceAccounts.
- [ ] Utilizar SecurityContext.
- [ ] Aplicar Pod Security Standards.
- [ ] Entender NetworkPolicies sob a ótica de segurança.
- [ ] Conhecer boas práticas para imagens, Secrets e Supply Chain.

---

# Pré-requisitos

- [[09 - ConfigMaps e Secrets]]
- [[15 - Networking no Kubernetes]]

---

# Modelo Mental

Imagine um prédio corporativo.

Existem diversas camadas de proteção.

```text
Portaria

↓

Crachá

↓

Elevador

↓

Sala

↓

Computador

↓

Aplicação
```

Mesmo que uma camada falhe, ainda existem outras.

No Kubernetes ocorre exatamente a mesma ideia.

Nunca confiamos em apenas um mecanismo.

---

# As camadas de segurança

Podemos dividir a segurança em sete camadas.

```text
Usuário

↓

Autenticação

↓

Autorização

↓

Admission Controllers

↓

Pod Security

↓

Rede

↓

Container

↓

Aplicação
```

Cada camada reduz a superfície de ataque.

---

# Autenticação (Authentication)

Primeira pergunta.

Quem está tentando acessar o cluster?

O Kubernetes suporta diversos métodos.

- Certificados X.509
- OpenID Connect (OIDC)
- Tokens
- ServiceAccounts
- Webhooks
- IAM (EKS, AKS, GKE)

---

# Autorização (Authorization)

Depois de identificar o usuário.

Surge outra pergunta.

> O que ele pode fazer?

Essa decisão é responsabilidade do mecanismo de autorização.

O mais utilizado é o RBAC.

---

# Authentication vs Authorization

Essa diferença costuma aparecer em entrevistas.

Authentication.

```text
Quem é você?
```

Authorization.

```text
O que você pode fazer?
```

---

# RBAC

Role-Based Access Control.

É o principal mecanismo de autorização do Kubernetes.

Ele responde perguntas como.

```text
Usuário João

↓

Pode listar Pods?
```

Ou.

```text
ServiceAccount API

↓

Pode acessar Secrets?
```

---

# Como o RBAC funciona?

Existem quatro recursos principais.

```text
Role

↓

RoleBinding

↓

ClusterRole

↓

ClusterRoleBinding
```

---

# Role

Define permissões.

Exemplo.

```text
Pode:

listar Pods

criar ConfigMaps

ler Deployments
```

Sempre dentro de um Namespace.

---

# ClusterRole

Mesmo conceito.

Mas válido para todo o cluster.

Exemplos.

```text
Nodes

PersistentVolumes

Namespaces
```

---

# RoleBinding

Associa.

```text
Usuário

↓

Role
```

---

# ClusterRoleBinding

Associa.

```text
Usuário

↓

ClusterRole
```

---

# Fluxo do RBAC

```text
Usuário

↓

API Server

↓

Authentication

↓

RBAC

↓

Permitido

ou

Negado
```

---

# ServiceAccounts

Até agora falamos de usuários.

Mas...

Quem representa uma aplicação?

A resposta é.

ServiceAccount.

---

# Modelo Mental

Imagine um funcionário.

Ele possui um crachá.

Uma aplicação também precisa de um.

Esse crachá é a ServiceAccount.

---

# ServiceAccount padrão

Todo Namespace possui.

```text
default
```

Se você não informar outra.

O Pod utilizará essa automaticamente.

---

# Boas práticas

Nunca utilize a ServiceAccount padrão em produção.

Crie uma específica para cada aplicação.

---

# Exemplo

```yaml
apiVersion: v1

kind: ServiceAccount

metadata:

  name: api-sa
```

Depois.

```yaml
serviceAccountName: api-sa
```

---

# SecurityContext

Uma das funcionalidades mais importantes.

Permite controlar como o container será executado.

---

# Executar como Root

Por padrão.

Muitas imagens executam como.

```text
root
```

Isso aumenta o impacto caso o container seja comprometido.

---

# runAsUser

Exemplo.

```yaml
securityContext:

  runAsUser: 1000
```

O processo não será executado como root.

---

# runAsNonRoot

Ainda melhor.

```yaml
runAsNonRoot: true
```

Se a imagem tentar iniciar como root.

O Pod nem sobe.

---

# readOnlyRootFilesystem

Outra excelente prática.

```yaml
readOnlyRootFilesystem: true
```

O filesystem torna-se somente leitura.

Reduzindo persistência de malware.

---

# allowPrivilegeEscalation

Outra configuração importante.

```yaml
allowPrivilegeEscalation: false
```

Impede que processos elevem privilégos.

---

# Linux Capabilities

No Linux.

Nem todo privilégio depende do usuário root.

Existem capacidades.

Exemplos.

```text
NET_ADMIN

SYS_ADMIN

SYS_TIME
```

Boa prática.

Remover todas.

```yaml
capabilities:

  drop:

  - ALL
```

Adicionar apenas o necessário.

---

# Privileged Containers

Evite.

```yaml
privileged: true
```

Esse modo concede praticamente acesso total ao host.

É extremamente perigoso.

---

# HostPath

Lembra do capítulo de Volumes?

HostPath monta diretórios do Node.

Isso pode expor arquivos críticos do sistema.

Utilize apenas quando realmente necessário.

---

# Pod Security Standards

O Kubernetes define três níveis.

---

## Privileged

Quase sem restrições.

Indicado apenas para componentes de infraestrutura.

---

## Baseline

Bloqueia configurações perigosas.

Boa escolha para a maioria dos workloads.

---

## Restricted

Máximo nível de proteção.

Exige diversas boas práticas.

É o nível recomendado para aplicações.

---

# Admission Controllers

Imagine.

Um desenvolvedor cria.

```yaml
privileged: true
```

Como impedir?

Antes do Pod ser criado.

Existe uma etapa.

```text
Admission
```

Ela pode validar ou modificar recursos.

---

# OPA Gatekeeper

Permite criar políticas.

Exemplo.

```text
Nenhum Pod pode executar como root.
```

---

# Kyverno

Alternativa extremamente popular.

Além de validar.

Também consegue modificar recursos automaticamente.

Exemplo.

Adicionar labels obrigatórias.

Ou.

Injetar SecurityContexts.

---

# NetworkPolicies

No capítulo anterior vimos o funcionamento técnico.

Agora pense nelas como firewall.

```text
Frontend

↓

API

↓

Banco
```

Permitido.

```text
Frontend

↓

Banco
```

Bloqueado.

---

# Secrets

Nunca.

```yaml
env:

- PASSWORD=123456
```

Sempre utilize.

```text
Secret
```

Mesmo assim.

Lembre-se.

Secrets são armazenados em Base64.

Base64 não é criptografia.

---

# Encryption at Rest

Boa prática.

Habilitar criptografia no etcd.

Assim.

Mesmo que alguém obtenha acesso ao disco.

Os Secrets continuam protegidos.

---

# Image Security

Nunca utilize.

```text
latest
```

Prefira.

```text
nginx:1.25.3
```

Ou melhor.

```text
Imagem por Digest
```

```text
sha256:...
```

---

# Image Scanning

Escaneie imagens antes do deploy.

Ferramentas populares.

- Trivy
- Grype
- Snyk
- Clair

---

# Supply Chain Security

Nos últimos anos.

Esse tornou-se um dos temas mais importantes.

Boas práticas.

- imagens assinadas;
- SBOM;
- Cosign;
- Sigstore;
- Provenance.

---

# Zero Trust

Nunca confie.

Nem mesmo dentro do cluster.

Cada comunicação deve ser explicitamente autorizada.

---

# Hands-on

Verificar ServiceAccounts.

```bash
kubectl get serviceaccounts
```

---

Verificar permissões.

```bash
kubectl auth can-i get pods
```

---

Verificar como um usuário específico.

```bash
kubectl auth can-i create deployments \
--as=system:serviceaccount:default:api-sa
```

---

Verificar SecurityContext.

```bash
kubectl describe pod api
```

---

Listar Roles.

```bash
kubectl get roles
```

---

Listar ClusterRoles.

```bash
kubectl get clusterroles
```

---

# Comparando os mecanismos

| Recurso | Função |
|----------|--------|
| Authentication | Identificar quem acessa |
| Authorization (RBAC) | Definir permissões |
| ServiceAccount | Identidade da aplicação |
| Admission Controller | Validar e modificar recursos |
| SecurityContext | Restringir privilégios do container |
| NetworkPolicy | Firewall entre Pods |
| Secrets | Armazenar dados sensíveis |

---

# Relação com os capítulos anteriores

Agora nossa arquitetura possui uma camada de segurança.

```text
Internet
      │
Ingress
      │
Service
      │
NetworkPolicy
      │
Pods
      │
SecurityContext
      │
Containers
      │
ServiceAccount
      │
RBAC
      │
API Server
      │
Authentication
```

---

# Boas práticas

- Nunca execute containers como root.
- Crie uma ServiceAccount por aplicação.
- Utilize RBAC com o princípio do menor privilégio (Least Privilege).
- Ative criptografia do etcd.
- Escaneie imagens continuamente.
- Utilize Pod Security Standards no nível Restricted quando possível.
- Implemente NetworkPolicies desde o início do projeto.
- Utilize Admission Controllers para aplicar políticas organizacionais.

---

# Erros comuns

> [!WARNING]

Executar todos os Pods utilizando a ServiceAccount `default`.

Isso dificulta o controle de permissões.

---

> [!WARNING]

Conceder permissões de `cluster-admin` para aplicações.

Esse é um dos erros mais críticos em ambientes Kubernetes.

---

> [!WARNING]

Executar containers privilegiados sem necessidade.

Isso pode permitir comprometimento do Node.

---

> [!WARNING]

Acreditar que Base64 protege Secrets.

Base64 é apenas um formato de codificação.

---

> [!WARNING]

Utilizar a tag `latest` em produção.

Isso reduz a reprodutibilidade e dificulta auditorias.

---

# Resumo

Neste capítulo aprendemos que:

- Segurança no Kubernetes é composta por múltiplas camadas.
- Authentication identifica usuários; Authorization define permissões.
- RBAC implementa controle de acesso baseado em papéis.
- ServiceAccounts representam aplicações.
- SecurityContext reduz privilégios dos containers.
- Pod Security Standards definem níveis de proteção.
- Admission Controllers aplicam políticas antes da criação dos recursos.
- NetworkPolicies controlam o tráfego entre Pods.
- Segurança da cadeia de fornecimento (Supply Chain) é parte essencial de ambientes modernos.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual a diferença entre Authentication e Authorization?
- [ ] O que é RBAC?
- [ ] Quando utilizar Role e ClusterRole?
- [ ] Qual o papel de uma ServiceAccount?
- [ ] O que faz o SecurityContext?
- [ ] Por que evitar containers privilegiados?
- [ ] O que são Pod Security Standards?
- [ ] Como proteger Secrets?
- [ ] O que é Supply Chain Security?
- [ ] Como aplicar o princípio de Zero Trust no Kubernetes?

---

# Diagrama consolidado

```text
                        Usuário / Aplicação
                               │
                      Authentication
                               │
                            RBAC
                               │
                   Admission Controllers
                               │
                 Pod Security Standards
                               │
                      ServiceAccount
                               │
                     SecurityContext
                               │
                     NetworkPolicies
                               │
                          Ingress
                               │
                           Service
                               │
                            Pod
                               │
                        Container
                               │
                          Secrets
                               │
                        Persistent Storage
```
