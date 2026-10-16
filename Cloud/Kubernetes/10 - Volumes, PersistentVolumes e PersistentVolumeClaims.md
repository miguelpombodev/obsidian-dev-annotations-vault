---
tags:
  - kubernetes
  - storage
  - volumes
  - persistentvolume
  - pvc
  - storageclass
  - stateful
---
> [!INFO] Objetivo
> Entender como o Kubernetes trata armazenamento, por que o sistema de arquivos de um container é efêmero e como funciona toda a arquitetura de persistência utilizando Volumes, PersistentVolumes (PV), PersistentVolumeClaims (PVC) e StorageClasses.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar por que containers são efêmeros.
- [ ] Entender o ciclo de vida dos Volumes.
- [ ] Diferenciar Volume, PV, PVC e StorageClass.
- [ ] Entender Provisionamento Estático e Dinâmico.
- [ ] Explicar como aplicações stateful armazenam dados.
- [ ] Escolher o tipo de armazenamento adequado para diferentes cenários.

---

# Pré-requisitos

- [[04 - Pods]]
- [[07 - Deployments]]
- [[09 - ConfigMaps e Secrets]]

---

# O problema que este capítulo resolve

Imagine uma API.

Ela salva arquivos em:

```text
/uploads
```

Ou um PostgreSQL gravando dados em:

```text
/var/lib/postgresql/data
```

Tudo funciona.

Até que...

```bash
kubectl delete pod postgres
```

Novo Pod criado.

Todos os dados desapareceram.

Por quê?

---

# Containers possuem filesystem efêmero

Todo container possui um sistema de arquivos.

```text
Container

↓

Filesystem
```

Esse filesystem existe apenas enquanto o container existe.

Quando o container morre.

```text
Filesystem

↓

Destruído
```

Tudo que estava nele desaparece.

---

# Modelo Mental

Imagine um computador alugado.

Você salva arquivos apenas na memória RAM.

Quando desliga.

Tudo desaparece.

O filesystem padrão do container funciona dessa forma.

---

# O problema dos Pods

Os Pods também são efêmeros.

Lembre-se.

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Se um Pod for recriado.

O novo Pod possui outro filesystem.

Outro container.

Outro disco.

Nada é reaproveitado.

---

# Como resolver?

Separando:

Aplicação

↓

Dados

Em vez de:

```text
Container

↓

Dados
```

Temos:

```text
Container

↓

Volume

↓

Disco
```

---

# O que é um Volume?

Um Volume é um diretório montado dentro do Pod.

Visualmente.

```text
Pod

├── Container

└── Volume
```

O Volume sobrevive ao reinício do container.

Mas nem sempre sobrevive ao Pod.

Depende do tipo de Volume.

---

# Tipos de armazenamento

Existem vários.

Os principais.

- emptyDir
- hostPath
- PersistentVolume
- CSI Volumes
- NFS
- Cloud Volumes

---

# emptyDir

É criado quando o Pod nasce.

```text
Criar Pod

↓

Criar Volume
```

Todos os containers compartilham esse Volume.

```text
Container A

↓

emptyDir

↑

Container B
```

Quando o Pod morre.

```text
Volume

↓

Destruído
```

Muito utilizado para:

- cache
- arquivos temporários
- processamento intermediário

---

# hostPath

Utiliza uma pasta do próprio Node.

```text
Worker Node

↓

/dados

↓

Pod
```

Problema.

Se o Pod mudar de Node.

Os dados ficam para trás.

Por isso.

É indicado apenas para:

- laboratórios
- Minikube
- Kind
- desenvolvimento

---

# PersistentVolume (PV)

Até agora falamos do Pod.

Agora vamos falar do Cluster.

Um PersistentVolume representa um armazenamento disponível para o cluster.

Exemplo.

```text
Disco AWS EBS

↓

PersistentVolume
```

Ou.

```text
Azure Disk

↓

PersistentVolume
```

Ou.

```text
NFS

↓

PersistentVolume
```

O PV representa um recurso físico ou lógico.

---

# PersistentVolumeClaim (PVC)

Aplicações não utilizam PersistentVolumes diretamente.

Elas fazem um pedido.

Esse pedido chama-se.

PersistentVolumeClaim.

Modelo mental.

```text
PersistentVolume

↓

Apartamento
```

```text
PersistentVolumeClaim

↓

Contrato de aluguel
```

A aplicação não escolhe o disco.

Ela apenas solicita.

```text
Preciso de:

20 GB

ReadWriteOnce
```

O Kubernetes encontra um PV compatível.

---

# Fluxo completo

```text
Pod

↓

PVC

↓

PV

↓

Disco
```

Esse fluxo é extremamente importante.

---

# Anatomia de um PV

```yaml
apiVersion: v1

kind: PersistentVolume

metadata:

  name: pv-postgres

spec:

  capacity:

    storage: 20Gi

  accessModes:

    - ReadWriteOnce
```

---

# Anatomia de um PVC

```yaml
apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: postgres-pvc

spec:

  accessModes:

    - ReadWriteOnce

  resources:

    requests:

      storage: 20Gi
```

Observe.

O PVC não informa qual disco deseja.

Ele informa apenas requisitos.

---

# Access Modes

Os principais são.

## ReadWriteOnce (RWO)

Um único Node pode montar o volume para leitura e escrita.

É o modo mais comum para bancos de dados.

---

## ReadOnlyMany (ROX)

Vários Nodes podem montar o volume apenas para leitura.

---

## ReadWriteMany (RWX)

Vários Nodes podem montar simultaneamente para leitura e escrita.

Muito utilizado com:

- NFS
- CephFS
- Azure Files
- Amazon EFS

---

# StorageClass

Até agora assumimos que o disco já existia.

Mas em Cloud isso normalmente não acontece.

Entra então a StorageClass.

Ela funciona como um template de provisionamento.

```text
PVC

↓

StorageClass

↓

Cloud Provider

↓

Novo Disco

↓

PV

↓

Pod
```

---

# Provisionamento Estático

Administrador cria o PV.

Depois.

Aplicação cria o PVC.

```text
PV

↓

PVC

↓

Pod
```

---

# Provisionamento Dinâmico

Mais comum em Cloud.

```text
PVC

↓

StorageClass

↓

Criar Disco

↓

Criar PV

↓

Associar
```

Tudo automático.

---

# Como funciona na AWS

Exemplo.

```text
PVC

↓

StorageClass

↓

EBS CSI Driver

↓

Criar volume EBS

↓

PersistentVolume

↓

Pod
```

Você nunca cria o disco manualmente.

---

# Montando um PVC

Deployment.

```yaml
volumes:

- name: dados

  persistentVolumeClaim:

    claimName: postgres-pvc
```

Depois.

```yaml
volumeMounts:

- mountPath: /var/lib/postgresql/data

  name: dados
```

Fluxo.

```text
Container

↓

/var/lib/postgresql/data

↓

PVC

↓

PV

↓

Disco
```

---

# Internals

Fluxo completo.

```text
Deployment

↓

Pod

↓

PVC

↓

StorageClass

↓

CSI Driver

↓

Cloud Provider

↓

Volume

↓

PV

↓

Mount

↓

Container
```

Observe quantas camadas existem.

Isso permite abstração.

A aplicação nunca sabe se o disco é:

- EBS
- Azure Disk
- Ceph
- NFS
- SAN

---

# CSI (Container Storage Interface)

Antigamente cada fornecedor implementava sua própria integração.

Hoje praticamente todos utilizam CSI.

Exemplos.

- AWS EBS CSI
- Azure Disk CSI
- GCE PD CSI
- Ceph CSI
- Longhorn CSI
- OpenEBS CSI

CSI tornou o Kubernetes independente do provedor de armazenamento.

---

# Hands-on

Criar PVC.

```yaml
apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: nginx-pvc

spec:

  accessModes:

    - ReadWriteOnce

  resources:

    requests:

      storage: 5Gi
```

Aplicar.

```bash
kubectl apply -f pvc.yaml
```

Verificar.

```bash
kubectl get pvc
```

Ver PV associado.

```bash
kubectl get pv
```

Descrever.

```bash
kubectl describe pvc nginx-pvc
```

---

# Como isso aparece na prática

Imagine um PostgreSQL.

```text
Deployment (ou StatefulSet)

↓

Pod

↓

PVC

↓

StorageClass

↓

AWS EBS

↓

Disco Persistente
```

Mesmo que o Pod seja recriado.

O disco continua existindo.

O novo Pod monta exatamente o mesmo volume.

Os dados permanecem intactos.

---

# Relação com os capítulos anteriores

Nosso modelo mental evolui.

```text
Deployment
      │
ReplicaSet
      │
Pods
 ┌────┴─────┐
 │          │
ConfigMap Secret
 │          │
 └────┬─────┘
      │
Volumes
      │
PVC
      │
PV
      │
StorageClass
      │
Cloud Provider
```

---

# Boas práticas

- Nunca armazene dados importantes no filesystem do container.
- Utilize PVCs em vez de hostPath.
- Prefira Provisionamento Dinâmico.
- Utilize StorageClasses fornecidas pelo ambiente.
- Escolha o AccessMode correto para a aplicação.

---

# Erros comuns

> [!WARNING]

Acreditar que reiniciar um Pod preserva o filesystem do container.

O filesystem do container é descartável.

---

> [!WARNING]

Utilizar hostPath em produção.

Esse tipo de volume cria forte acoplamento ao Node.

---

> [!WARNING]

Criar aplicações stateful utilizando apenas Deployments sem persistência.

Os dados serão perdidos quando os Pods forem recriados.

---

> [!WARNING]

Confundir PVC com PV.

O PVC é uma solicitação.

O PV representa o armazenamento disponível.

---

# Resumo

Neste capítulo aprendemos que:

- Containers possuem armazenamento efêmero.
- Volumes desacoplam dados do ciclo de vida do container.
- PersistentVolumes representam recursos de armazenamento.
- PersistentVolumeClaims solicitam armazenamento.
- StorageClasses automatizam o provisionamento.
- CSI padroniza a integração entre Kubernetes e provedores de armazenamento.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Por que o filesystem do container é efêmero?
- [ ] Qual a diferença entre Volume, PV e PVC?
- [ ] O que faz uma StorageClass?
- [ ] O que é Provisionamento Dinâmico?
- [ ] Quando utilizar ReadWriteMany?
- [ ] Qual o papel do CSI?

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
          ┌──────────┼──────────┐
          │          │          │
     ConfigMap    Secret     Volumes
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
