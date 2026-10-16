---
tags:
  - kubernetes
  - configmap
  - secrets
  - configuration
  - twelve-factor
---
> [!INFO] Objetivo
> Entender como o Kubernetes gerencia configurações e informações sensíveis, por que elas não devem fazer parte da imagem Docker e quais são as melhores práticas para aplicações cloud-native.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Entender por que configurações não pertencem à imagem Docker.
- [ ] Explicar o conceito de ConfigMap.
- [ ] Explicar o conceito de Secret.
- [ ] Entender as diferentes formas de consumir ConfigMaps e Secrets.
- [ ] Diferenciar informações públicas de informações sensíveis.
- [ ] Aplicar boas práticas de gerenciamento de configuração.

---

# Pré-requisitos

- [[04 - Pods]]
- [[07 - Deployments]]
- [[08 - Services]]

---

# O problema que este capítulo resolve

Imagine que você possui uma API.

Ela precisa das seguintes informações.

```text
HOST

PORT

DATABASE_URL

REDIS_HOST

JWT_SECRET

API_KEY

LOG_LEVEL
```

Onde colocar tudo isso?

Algumas pessoas fazem assim.

```python
DATABASE_URL = "postgres://..."
JWT_SECRET = "123456"
```

Ou.

```dockerfile
ENV DATABASE_URL=...
ENV JWT_SECRET=...
```

Ou pior.

```python
senha = "MinhaSenhaSuperSecreta"
```

Todos esses exemplos possuem problemas.

---

# O grande problema

Imagine três ambientes.

```text
Desenvolvimento
```

```text
Homologação
```

```text
Produção
```

A aplicação é exatamente a mesma.

O que muda?

A configuração.

Não o código.

---

# Twelve-Factor App

Um dos princípios diz:

> Configuração deve ficar fora do código.

Isso significa.

Código.

```text
↓

Imagem Docker
```

Configuração.

```text
↓

Kubernetes
```

Credenciais.

```text
↓

Secrets
```

---

# Modelo Mental

Imagine um notebook.

O notebook representa a aplicação.

Os post-its representam a configuração.

Você pode trocar os post-its.

O notebook continua o mesmo.

---

# Configuração versus Estado

É importante diferenciar.

Configuração.

```text
Porta

Host

Feature Flags

Timeouts
```

Estado.

```text
Banco

Arquivos

Pedidos

Clientes
```

ConfigMaps armazenam configuração.

Não armazenam estado.

---

# O que é um ConfigMap?

Um ConfigMap é um objeto da API que armazena pares chave/valor.

Exemplo.

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: api-config

data:

  LOG_LEVEL: INFO

  PORT: "8080"

  FEATURE_X: "true"
```

Observe.

Tudo é armazenado como texto.

---

# Como um ConfigMap funciona?

Fluxo.

```text
ConfigMap

↓

API Server

↓

etcd

↓

Pod

↓

Container
```

O Pod pode consumir essas informações.

---

# Formas de consumir um ConfigMap

Existem três formas principais.

- Variáveis de ambiente
- Volume
- Argumentos de inicialização

---

# Como variável de ambiente

Exemplo.

```yaml
env:

- name: LOG_LEVEL

  valueFrom:

    configMapKeyRef:

      name: api-config

      key: LOG_LEVEL
```

Na aplicação Python.

```python
import os

log_level = os.getenv("LOG_LEVEL")
```

---

# Importando todas as variáveis

Também podemos fazer.

```yaml
envFrom:

- configMapRef:

    name: api-config
```

Todas as chaves viram variáveis de ambiente.

---

# Montando como arquivos

Outra possibilidade.

```yaml
volumes:

- name: config

  configMap:

    name: api-config
```

Depois.

```yaml
volumeMounts:

- mountPath: /config

  name: config
```

Resultado.

```text
/config

LOG_LEVEL

PORT

FEATURE_X
```

Cada chave vira um arquivo.

---

# Quando utilizar Volume?

Muito comum para.

- nginx.conf
- prometheus.yml
- grafana.ini
- application.properties
- config.json

Ou qualquer arquivo de configuração.

---

# O que é um Secret?

Um Secret é muito parecido com um ConfigMap.

A diferença é conceitual.

Ele armazena informações sensíveis.

Por exemplo.

```text
Senhas

Tokens

JWT Keys

TLS Certificates

SSH Keys

API Keys
```

---

# Anatomia de um Secret

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: api-secret

type: Opaque

stringData:

  DATABASE_PASSWORD: MinhaSenha

  JWT_SECRET: abc123
```

Observe.

Estamos utilizando.

```yaml
stringData
```

O Kubernetes fará a conversão automaticamente.

---

# Base64

Muitos exemplos utilizam.

```yaml
data:
```

Nesse caso.

Os valores devem estar em Base64.

Exemplo.

```text
admin
```

↓

```text
YWRtaW4=
```

Mas atenção.

> Base64 não é criptografia.

É apenas codificação.

Qualquer pessoa pode fazer.

```bash
echo YWRtaW4= | base64 -d
```

Resultado.

```text
admin
```

---

# O Secret é realmente seguro?

Depende.

Por padrão.

Os Secrets ficam armazenados no etcd.

Se o etcd não estiver criptografado.

Os dados poderão ser recuperados.

Em produção.

O recomendado é habilitar.

Encryption at Rest.

---

# Consumindo Secrets

Exatamente igual ao ConfigMap.

Variável de ambiente.

```yaml
env:

- name: DATABASE_PASSWORD

  valueFrom:

    secretKeyRef:

      name: api-secret

      key: DATABASE_PASSWORD
```

Ou.

Volume.

```yaml
volumes:

- secret:

    secretName: api-secret
```

---

# Fluxo completo

```text
Secret

↓

API Server

↓

etcd

↓

Pod

↓

Volume

↓

Aplicação
```

---

# ConfigMap x Secret

| ConfigMap | Secret |
|------------|---------|
| Configurações | Informações sensíveis |
| Texto simples | Dados sensíveis |
| LOG_LEVEL | Senhas |
| PORT | Tokens |
| TIMEOUT | Certificados |

---

# Hands-on

Criar ConfigMap.

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: app-config

data:

  APP_NAME: ecommerce

  LOG_LEVEL: DEBUG
```

Aplicar.

```bash
kubectl apply -f configmap.yaml
```

Consultar.

```bash
kubectl get configmaps
```

Detalhes.

```bash
kubectl describe configmap app-config
```

---

Criar Secret.

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: app-secret

type: Opaque

stringData:

  DATABASE_PASSWORD: minhaSenha
```

Aplicar.

```bash
kubectl apply -f secret.yaml
```

Consultar.

```bash
kubectl get secrets
```

---

# Como isso aparece na prática

Imagine uma API em produção.

Ela utiliza.

```text
Deployment
```

↓

```text
ConfigMap

↓

LOG_LEVEL

TIMEOUT

HOST

PORT
```

↓

```text
Secret

↓

DATABASE_PASSWORD

JWT_SECRET

API_KEY
```

A imagem Docker permanece exatamente a mesma.

Apenas a configuração muda entre ambientes.

---

# Integração com provedores de segredo

Em ambientes corporativos, raramente armazenamos segredos diretamente no Kubernetes.

É comum utilizar.

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

O Kubernetes acessa esses sistemas por meio de operadores, CSI Drivers ou soluções como o **External Secrets Operator**, sincronizando os segredos para o cluster.

---

# Relação com os capítulos anteriores

Agora nosso modelo mental evolui.

```text
                    ConfigMap
                        │
                        ▼
Deployment ───► Pods ◄── Secret
       │            │
       ▼            ▼
 ReplicaSet    Containers
       ▲
       │
     Service
       ▲
       │
    Cliente
```

Os Pods agora recebem configuração e credenciais externas, sem necessidade de reconstruir a imagem.

---

# Boas práticas

- Nunca armazene senhas na imagem Docker.
- Utilize ConfigMaps apenas para configurações não sensíveis.
- Utilize Secrets para credenciais.
- Versione os manifestos, mas nunca exponha valores sensíveis em repositórios públicos.
- Prefira integrar o Kubernetes a um gerenciador de segredos externo em produção.
- Evite utilizar `latest` como imagem quando mudanças de configuração dependem de versões específicas da aplicação.

---

# Erros comuns

> [!WARNING]

Colocar senhas em ConfigMaps.

ConfigMaps não foram projetados para informações sensíveis.

---

> [!WARNING]

Acreditar que Base64 protege informações.

Base64 apenas codifica dados.

---

> [!WARNING]

Alterar um ConfigMap esperando que a aplicação recarregue automaticamente.

Na maioria dos casos, aplicações que consomem configuração via variável de ambiente precisam ser reiniciadas para enxergar a nova configuração.

---

> [!WARNING]

Versionar Secrets com valores reais em repositórios Git.

Mesmo em repositórios privados, isso aumenta significativamente o risco de exposição.

---

# Resumo

Neste capítulo aprendemos que:

- Configuração deve ser separada do código.
- ConfigMaps armazenam configurações.
- Secrets armazenam informações sensíveis.
- Ambos podem ser consumidos como variáveis de ambiente ou volumes.
- Base64 não é criptografia.
- Em produção, é recomendável integrar o cluster a um gerenciador de segredos dedicado.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que configurações não devem ficar na imagem Docker?
- [ ] Qual a diferença entre ConfigMap e Secret?
- [ ] Quais são as formas de consumir ConfigMaps?
- [ ] O que significa `stringData` em um Secret?
- [ ] Por que Base64 não é um mecanismo de segurança?
- [ ] Quais soluções externas podem ser utilizadas para gerenciar segredos?

---

# Diagrama consolidado

```text
                     Cliente
                        │
                    CoreDNS
                        │
                     Service
                        │
                   EndpointSlice
                        │
                    kube-proxy
                        │
                    Deployment
                        │
                    ReplicaSet
                        │
                       Pods
                  ┌─────┴─────┐
                  │           │
            ConfigMap     Secret
                  │           │
                  └─────┬─────┘
                        ▼
                   Containers
```

