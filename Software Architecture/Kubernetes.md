
# 🧠 Introdução ao Kubernetes — Guia Avançado e Completo para Estudantes de Tecnologia

Kubernetes (também chamado de K8s) é uma plataforma open source que automatiza o deploy, escalonamento e gerenciamento de aplicações baseadas em containers (como os que criamos com Docker). Este guia abrangente e avançado oferece uma compreensão profunda de todos os aspectos do Kubernetes para estudantes e profissionais de tecnologia.

## ☁️ Breve História do Kubernetes

O Kubernetes foi criado pelo Google com base em sua experiência anterior com um sistema interno chamado Borg, depois evoluído para o Omega. O projeto foi então lançado publicamente com o nome de Kubernetes e hoje é mantido pela Cloud Native Computing Foundation (CNCF).

É por isso que o Google Cloud Platform (GCP) tem um suporte muito bom para Kubernetes — eles foram pioneiros na i📌 
Pontos-Chave Fundamentais

- O Kubernetes é controlado por uma **API poderosa**
- A principal ferramenta de linha de comando para interagir com o Kubernetes é o **kubectl**
- Trabalhamos de forma **declarativa**, ou seja, dizemos "o que" queremos (e não "como") através de arquivos YAML
- O Kubernetes funciona dentro de um **cluster**, que é um grupo de máquinas (ou nós, "nodes") conectadas

## 🔧 Componentes Fundamentais Detalhados

## 🎯 Arquitetura do Cluster

Um cluster Kubernetes é um conjunto de máquinas (VMs ou instâncias físicas) organizadas em dois grupos principais:

- **Control Plane (Master Node)**: onde a lógica e controle central do Kubernetes acontece
- **Worker Nodes**: onde os containers e aplicações rodam

## 🧠 Control Plane – O Cérebro do Sistema

O control plane é composto por vários componentes críticos que trabalham em conjunto:

## **kube-apiserver** - O Coração do Sistema

O kube-apiserver é o componente mais crítico do control plane. Suas responsabilidades incluem:

- **Ponto de entrada da API**: Tudo que você manda pelo kubectl passa por aqui

**Validação e autenticação**: Verifica identidade, autorização e validação de requests

- **Controle de admissão**: Executa policies através de admission controllers
- **Comunicação com etcd**: É o único componente que fala diretamente com o etcd
- **Processamento RESTful**: Fornece endpoints REST para todos os recursos Kubernetes

O kube-apiserver é composto por três servidores internos:

- **AggregatorServer**: Lida com APIs agregadas de terceiros
- **KubeAPIServer**: Processa recursos nativos do Kubernetes
- **APIExtensionServer**: Gerencia Custom Resource Definitions (CRDs)

## **etcd** - O Banco de Dados Distribuído

O etcd é um banco de dados chave-valor distribuído que serve como a única fonte de verdade para todo o estado do cluster.

**Características principais**:

- **Consistência forte**: Usa o protocolo Raft para consenso distribuído
- **Alta disponibilidade**: Funciona em clusters com múltiplos nós
- **Versionamento**: Mantém histórico de mudanças usando MVCC (Multi-Version Concurrency Control)
- **Compactação automática**: Remove versões antigas para economizar espaço

**O que o etcd armazena**:

- Estados de todos os objetos Kubernetes (pods, services, deployments, etc.)
- Configurações do cluster
- Secrets e ConfigMaps
- Metadados de rede e políticas

## **kube-scheduler** - O Orquestrador de Pods

O kube-scheduler é responsável por decidir em qual nó cada novo pod será executado.

**Processo de scheduling**:

1. **Filtering**: Remove nós que não atendem aos requisitos do pod
2. **Scoring**: Pontua os nós restantes com base em critérios como:
    - Disponibilidade de recursos (CPU, memória)
    - Afinidade e anti-afinidade
    - Taints e tolerations
    - Localização de dados
    - Políticas de qualidade de serviço

**Fatores considerados no scheduling**:

- **Node Affinity**: Preferências sobre onde o pod deve rodar
- **Pod Affinity/Anti-affinity**: Relacionamento entre pods
- **Taints e Tolerations**: Restrições específicas de nós
- **Resource requests e limits**: Necessidades de recursos do pod

## **kube-controller-manager** - O Gerenciador de Estado

O controller manager roda vários controladores que garantem que o estado atual do cluster seja igual ao desejado.

**Controladores principais**:

- **Node Controller**: Monitora a saúde dos nós
- **Job Controller**: Gerencia jobs que executam uma única vez
- **Endpoint Controller**: Popula objetos Endpoints
- **Service Account Controller**: Cria ServiceAccounts padrão para namespaces
- **Deployment Controller**: Gerencia ReplicaSets
- **ReplicaSet Controller**: Mantém o número correto de réplicas

## **cloud-controller-manager** - Integração com Nuvem

Este componente contém lógica específica do provedor de nuvem. Existe apenas quando executando em provedores de nuvem e inclui:

- **Node Controller**: Verifica com o provedor se um nó foi deletado
- **Route Controller**: Configura rotas na infraestrutura de nuvem
- **Service Controller**: Cria, atualiza e deleta load balancers

## 💪 Worker Nodes – Onde a Aplicação Roda

Os worker nodes executam as cargas de trabalho e contêm três componentes principais:

## **kubelet** - O Agente Principal

O kubelet é o agente primário que roda em cada nó worker.

**Responsabilidades do kubelet**:

- **Registro do nó**: Se registra no kube-apiserver como um recurso Node
- **Gerenciamento de pods**: Cria, modifica e deleta containers conforme especificado
- **Health checks**: Executa liveness, readiness e startup probes
- **Volume management**: Monta volumes lendo configurações do pod
- **Relatórios de status**: Reporta status do nó e pods via chamadas para o API server

**Interfaces utilizadas**:

- **CRI (Container Runtime Interface)**: Para comunicar com o container runtime
- **CSI (Container Storage Interface)**: Para configurar volumes de bloco
- **CNI (Container Network Interface)**: Para configurar rede dos pods

## **kube-proxy** - O Proxy de Rede

O kube-proxy mantém regras de rede nos nós e implementa a abstração de Service.

**Funcionalidades**:

- **Load balancing**: Distribui tráfego entre pods de um service
- **Service discovery**: Permite que pods se comuniquem via nomes de service
- **Implementações**: Pode usar iptables, IPVS ou userspace para roteamento

## **Container Runtime** - O Motor dos Containers

O container runtime é responsável por executar os containers.

**Principais opções**:

**Docker**: A solução mais conhecida, mas foi depreciada como runtime padrão

- Usa containerd como runtime de baixo nível
- Interface familiar para desenvolvedores
- Ainda funciona através do CRI-dockerd

**containerd**: Runtime industry-standard

- Leve e eficiente
- Projeta graduado pela CNCF
- Foco em simplicidade e robustez
- Interface direta com CRI

**CRI-O**: Runtime específico para Kubernetes

- Compliance total com OCI (Open Container Initiative)
- Designed especificamente para Kubernetes
- Menor superfície de ataque
- Usado no Red Hat OpenShift

## 📦 Objetos Kubernetes Fundamentais

## Pods – A Menor Unidade Executável

Um Pod é o menor e mais simples objeto no Kubernetes. Características importantes:

- **Compartilhamento de recursos**: Containers no mesmo pod compartilham:
    - IP interno
    - Sistema de arquivos (volumes)
    - Ciclo de vida
- **Padrão comum**: Geralmente um container por Pod
- **Ephemeral**: Pods são substituíveis, não persistentes

![[KUB_Untitled.png]]

## Deployments e ReplicaSets - Gerenciamento de Réplicas

**Deployment** é um objeto que descreve o estado desejado para pods:

- Quantas réplicas de um Pod queremos
- Qual imagem de container usar
- Como lidar com atualizações (Rolling Updates, Blue-Green, etc.)

**ReplicaSet** é usado internamente pelo Deployment para manter a quantidade correta de réplicas.

**Exemplo de Deployment**:

```bash
**apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:1.21.6  # Sempre use tags específicas!
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"**
```

### Exemplos visuais:

**3 Backends e 2 Frontends:**

![[KUB_Untitled 1.png]]

3 Frontends:

![[KUB_Untitled 2.png]]

**Excedendo os limites de recursos:**

Se os nós do cluster não tiverem **CPU ou memória suficiente**, os pods ficam em estado **Pending**.

![[KUB_Untitled 3.png]]

# ⚙️ Comandos `kubectl` essenciais

```bash
kubectl get pods               # Lista os pods
kubectl get services           # Lista os serviços
kubectl describe pod <nome>    # Mostra detalhes de um pod
kubectl apply -f arquivo.yaml  # Aplica um arquivo de configuração
kubectl delete pod <nome>      # Deleta um pod específico
kubectl logs <nome-do-pod>     # Visualiza os logs do pod
```

## 🧠 Conceitos Importantes que Você PRECISA Saber

- **Namespaces**: usados para isolar recursos dentro do cluster.
- **Services**: expõem Pods para dentro ou fora do cluster (ClusterIP, NodePort, LoadBalancer).
- **Volumes**: armazenamento persistente para os Pods.
- **ConfigMap e Secret**: armazenam configurações e dados sensíveis.
- **Health Checks**: `livenessProbe` e `readinessProbe` para verificar o estado dos containers.
- **Horizontal Pod Autoscaler (HPA)**: aumenta ou diminui a quantidade de Pods com base em uso de CPU/memória.

## 🧪 Ferramentas para Desenvolvimento Local Avançado

## ✅ Minikube - O Veterano Confiável

Minikube simula um cluster Kubernetes completo em sua máquina local.

**Vantagens**:

- Suporte a múltiplos drivers (VirtualBox, VMware, Docker, etc.)
- Addons integrados (dashboard, ingress, metrics-server)
- Ambiente mais próximo da produção
- Suporte oficial do Kubernetes

**Desvantagens**:

- Maior consumo de recursos
- Apenas single-node por padrão
- Startup mais lento

**Comandos essenciais**:

```bash
# Iniciar cluster
minikube start --driver=docker --memory=4096 --cpus=2

# Habilitar addons
minikube addons enable ingress
minikube addons enable dashboard
minikube addons enable metrics-server

# Acessar dashboard
minikube dashboard

# Tunelamento para LoadBalancer
minikube tunnel

# Parar cluster
minikube stop
```

## ✅ Kind (Kubernetes IN Docker) - O Especialista em CI/CD

Kind cria clusters Kubernetes usando containers Docker como nodes.

**Vantagens**:

- Muito rápido para start/stop
- Suporte nativo a multi-node
- Ideal para testes automatizados e CI/CD
- Configuração via arquivo YAML

**Configuração multi-node**:

```bash
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
```

**Comandos**:

```bash
# Criar cluster com configuração
kind create cluster --config=kind-config.yaml --name=dev-cluster

# Listar clusters
kind get clusters

# Deletar cluster
kind delete cluster --name=dev-cluster
```

## ✅ K3s - O Minimalista Eficiente

K3s é uma distribuição leve do Kubernetes, ideal para edge computing e recursos limitados.

**Características**:

- Binário único de ~50MB
- Banco de dados SQLite por padrão (pode usar etcd)
- Traefik como ingress controller padrão
- Servicelb como load balancer integrado

**Instalação e uso**:

```bash
# Instalação simples
curl -sfL https://get.k3s.io | sh -

# Via k3d (K3s in Docker)
k3d cluster create mycluster --agents 2
k3d cluster list
k3d cluster delete mycluster
```

## **NodePort** - Exposição Básica Externa

Expõe o service numa porta específica de todos os nós.

**Características**:

- Porta entre 30000-32767
- Acessível via `<NodeIP>:<NodePort>`
- Útil para desenvolvimento e testes

```bash
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

## **LoadBalancer** - Produção Cloud-Native

Provisiona um load balancer externo no provedor de nuvem.

**Características**:

- IP público fornecido pelo provedor
- Alta disponibilidade
- Integração com serviços cloud (AWS ALB, GCP LB, Azure LB)

```bash
apiVersion: v1
kind: Service
metadata:
  name: production-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: production-app
  ports:
  - port: 80
    targetPort: 8080
```

## Ingress Controllers - Roteamento HTTP/HTTPS Avançado

**NGINX Ingress Controller** é uma das implementações mais populares.

**Instalação via Helm**:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

**Exemplo de Ingress com SSL**:

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**Traefik**

como alternativa moderna:

- Descoberta automática de serviços
- Dashboard web integrado
- Suporte nativo a Let's Encrypt
- Métricas Prometheus integradas

## 💾 Armazenamento e Configuração

## ConfigMaps - Configuração Externa

ConfigMaps armazenam dados de configuração não-confidenciais.

**Criação imperativa**:

```bash
# A partir de arquivos
kubectl create configmap app-config --from-file=config/

# A partir de literais
kubectl create configmap app-env \
  --from-literal=LOG_LEVEL=debug \
  --from-literal=DATABASE_URL=postgresql://...
```

**Uso como variáveis de ambiente**:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - configMapRef:
        name: app-env
```

**Uso como volume**:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

## Secrets - Dados Sensíveis

Secrets armazenam informações confidenciais em formato base64

.

**Tipos de Secrets**:

- `Opaque`: dados arbitrários definidos pelo usuário
- `kubernetes.io/service-account-token`: tokens de service account
- `kubernetes.io/dockercfg`: credenciais Docker
- `kubernetes.io/tls`: certificados TLS

**Exemplo de Secret TLS**:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi... # base64 encoded cert
  tls.key: LS0tLS1CRUdJTi... # base64 encoded key
```

## Volumes - Persistência de Dados

Kubernetes oferece múltiplos tipos de volumes:

**EmptyDir**: Volume temporário

```bash
volumes:
- name: temp-storage
  emptyDir:
    sizeLimit: 1Gi
```

**HostPath**: Monta diretório do host

```bash
volumes:
- name: host-storage
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

**PersistentVolume**: Armazenamento durável

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /data/mysql
```

## 🏥 Health Checks e Observabilidade

## Probes - Verificações de Saúde

Kubernetes oferece três tipos de probes para monitorar a saúde dos containers.

## **Liveness Probe** - Detecta Containers Mortos

Determina quando reiniciar um container.

```bash
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

## **Readiness Probe** - Controla Tráfego

Determina quando o container está pronto para receber tráfego.

```bash
readinessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - "pg_isready -U $POSTGRES_USER -d $POSTGRES_DB"
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

## **Startup Probe** - Containers com Inicialização Lenta

Para aplicações que demoram para inicializar.

```bash
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

## Monitoramento com Prometheus e Grafana

**Instalação do stack completo**:

```bash
# Adicionar repositórios Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Instalar kube-prometheus-stack (inclui Prometheus, Grafana, AlertManager)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=15d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi
```

**Métricas importantes para monitorar**:

- **Cluster**: CPU, memória, disco, rede dos nós
- **Pods**: Resource utilization, restart count, estado
- **Aplicação**: Latência, throughput, error rate
- **Kubernetes API**: Request rate, latência, errors

## ⚙️ kubectl - Comandos Avançados

## Comandos Essenciais para Produção

**Debugging avançado**:

```bash
# Debug de pods
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl logs -f <pod-name> -c <container-name>

# Execução interativa
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -c <container-name> -- <command>

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Proxy para API server
kubectl proxy --port=8080
```

**Gerenciamento de recursos**:

```bash
# Aplicar com validação
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --validate=true

# Rollout management
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Scaling
kubectl scale deployment nginx-deployment --replicas=5
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10
```

**Comandos para troubleshooting**:

```bash
# Top resources
kubectl top nodes
kubectl top pods --all-namespaces
kubectl top pods --containers

# Events do cluster
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector type=Warning

# Informações de contexto
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
```

## 🛠️ Ferramentas do Ecossistema

## Helm - Gerenciador de Pacotes

Helm é o gerenciador de pacotes padrão para Kubernetes.

**Comandos básicos**:

```bash
# Adicionar repositório
helm repo add bitnami https://charts.bitnami.com/bitnami

# Buscar charts
helm search repo postgres

# Instalar com valores customizados
helm install my-postgres bitnami/postgresql \
  --set auth.postgresPassword=secretpassword \
  --set primary.persistence.size=20Gi

# Upgrade
helm upgrade my-postgres bitnami/postgresql --reuse-values

# Rollback
helm rollback my-postgres 1
```

## Kustomize - Configuração Declarativa

Kustomize permite customizar configurações Kubernetes sem templates.

**Estrutura de diretórios**:

```bash
app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── development/
    │   ├── kustomization.yaml
    │   └── resources.yaml
    └── production/
        ├── kustomization.yaml
        └── resources.yaml
```

**Base kustomization.yaml**:

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: myapp
  version: v1
```

## ArgoCD - GitOps

ArgoCD implementa GitOps para Kubernetes.

**Instalação**:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Obter senha admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**Application manifest**:

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-k8s
    targetRevision: HEAD
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 🔒 Boas Práticas de Segurança e Produção

## Resource Management

**Sempre defina requests e limits**:

```bash
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

## Security Best Practices

**RBAC (Role-Based Access Control)**:

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Security Context**:

```bash
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
```

**Network Policies**

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## 🧠 Conceitos Must-Know Avançados

## Namespaces - Isolamento Lógico

Namespaces proporcionam isolamento de recursos:

```bash
# Criar namespace
kubectl create namespace production

# Configurar ResourceQuota
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
EOF
```

## Horizontal Pod Autoscaler (HPA)

Escalonamento automático baseado em métricas:

```bash
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## StatefulSets - Aplicações Stateful

Para aplicações que precisam de identidade estável:

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database-service
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: postgres:13
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

## ✅ Boas Práticas Avançadas

## Deployment Strategies

**Rolling Updates**:

```bash
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

**Blue-Green Deployment** via services e labels

**Canary Deployment** com Flagger ou Argo Rollouts

## Image Security

- **Nunca use `latest`** como tag da imagem
- Use **image scanning** (Trivy, Clair)
- Implemente **admission controllers** para validar imagens
- Use **multi-stage builds** para reduzir superfície de ataque

## Observabilidade Completa

**Logging estruturado**:

```bash
# Fluent Bit DaemonSet para coleta de logs
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

**Distributed Tracing** com Jaeger ou Zipkin

**Metrics customizadas** expostas via Prometheus format

## 🛑 Cuidados e Armadilhas Comuns

## Problemas de Resource Management

- **Pods sem limits** podem consumir recursos indefinidamente
- **Requests muito baixos** podem causar over-scheduling
- **QoS Classes** importantes: Guaranteed > Burstable > BestEffort

## Networking Issues

- **DNS resolution** pode falhar sem Service
- **Network Policies** podem bloquear tráfego inadvertidamente
- **LoadBalancer** pode ser caro - considere Ingress

## Storage Pitfalls

- **EmptyDir** é perdido quando Pod morre
- **hostPath** não funciona em clusters multi-node
- **PVC** pode ficar "bound" mesmo após deleção do Pod

## Security Gotchas

- **Default ServiceAccount** tem permissões desnecessárias
- **Secrets** não são criptografados em etcd por padrão
- **Root containers** são um risco de segurança

## 🛑 Cuidado com...

- Pods **não são persistentes** — evite gravar dados neles.
- Expor tudo com `NodePort` pode ser perigoso — prefira LoadBalancer ou Ingress.
- Monitoramento é essencial: sempre tenha visibilidade com ferramentas como **Prometheus**, **Grafana**, **Jaeger**.

## 🚀 Próximos Passos Avançados

## 1. Service Mesh

Estude **Istio** ou **Linkerd** para:

- Observabilidade automática
- Traffic management avançado
- Security policies
- Circuit breaking

## 2. GitOps Avançado

Implemente com **ArgoCD** ou **Flux**:

- Multi-cluster management
- Progressive delivery
- Policy as Code

## 3. Kubernetes Operators

Desenvolva **Custom Controllers**:

- Custom Resource Definitions (CRDs)
- Operator SDK
- Kubernetes API programming

## 4. Cluster Management

**Production-ready clusters**:

- Multi-master setup
- Backup e disaster recovery
- Cluster upgrades
- Certificate management

## 5. Advanced Workloads

- **Jobs e CronJobs** para tarefas batch
- **DaemonSets** para serviços de sistema
- **Custom Schedulers** para casos específicos

## 🤓 Conclusão

O Kubernetes parece complexo no início, mas **não desanime**. Ele é poderoso justamente porque resolve problemas reais de aplicações modernas em produção, como:

- escalar horizontalmente,
- gerenciar atualizações sem downtime,
- isolar componentes de forma segura,
- automatizar tudo com arquivos YAML.

Aos poucos, com prática e exemplos, você vai dominar isso. 💪

O Kubernetes é uma plataforma complexa mas extremamente 
poderosa que resolve problemas reais de aplicações modernas em produção.
 Este guia avançado cobriu:

- **Arquitetura detalhada** dos componentes do control plane e worker nodes
- **Ferramentas locais** para desenvolvimento (Minikube, Kind, K3s)
- **Networking avançado** com Services e Ingress
- **Armazenamento e configuração** com Volumes, ConfigMaps e Secrets
- **Health checks** e observabilidade com Prometheus/Grafana
- **kubectl avançado** para troubleshooting e automação
- **Ferramentas do ecossistema** (Helm, Kustomize, ArgoCD)
- **Boas práticas de segurança** e produção
- **Conceitos avançados** como HPA, StatefulSets e Network Policies

Lembre-se: a jornada no Kubernetes é contínua. Comece 
com os fundamentos, pratique em ambientes seguros, e evolua gradualmente
 para cenários mais complexos. A comunidade Kubernetes é vasta e 
acolhedora - não hesite em participar e contribuir!

**Recursos para continuar aprendendo**:

- Documentação oficial: kubernetes.io
- CNCF Landscape: landscape.cncf.io
- Kubernetes Slack: kubernetes.slack.com
- KubeCon + CloudNativeCon (conferências anuais)
- Certificações: CKA, CKAD, CKS

Com dedicação e prática, você dominará esta tecnologia fundamental para o futuro das aplicações cloud-native! 💪

## 🚀 Próximos passos

- Instale o **Minikube** ou o **Kind** e comece a aplicar comandos simples.
- Pratique com exemplos reais (como rodar um app .NET ou Node.js).
- Estude ferramentas que integram bem com Kubernetes como:
    - Helm (gerenciador de pacotes para Kubernetes)
    - Kustomize
    - ArgoCD (GitOps)
    - Lens (UI para cluster)