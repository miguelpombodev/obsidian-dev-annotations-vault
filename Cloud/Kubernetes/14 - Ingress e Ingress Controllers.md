---
tags:
  - kubernetes
  - ingress
  - ingress-controller
  - nginx
  - traefik
  - gateway-api
  - networking
  - tls
---
> [!INFO] Objetivo
> Entender como aplicações HTTP/HTTPS são expostas para o mundo externo, compreender a diferença entre o recurso Ingress e um Ingress Controller e aprender como funciona todo o fluxo de entrada de tráfego em um cluster Kubernetes.

---

# Objetivos de aprendizagem

Ao concluir este capítulo você deverá ser capaz de:

- [ ] Explicar por que Services não são suficientes para aplicações HTTP.
- [ ] Diferenciar Ingress de Ingress Controller.
- [ ] Entender como ocorre o roteamento HTTP.
- [ ] Configurar múltiplos domínios utilizando um único Load Balancer.
- [ ] Compreender TLS, certificados e HTTPS no Kubernetes.
- [ ] Entender por que a Gateway API está substituindo o Ingress.

---

# Pré-requisitos

- [[08 - Services]]
- [[12 - DaemonSets]]
- Conhecimentos básicos de HTTP, DNS e TLS

---

# O problema que este capítulo resolve

Imagine um cluster contendo quatro aplicações.

```text
Frontend

API

Auth

Admin
```

Cada aplicação possui um Service.

```text
Frontend Service

API Service

Auth Service

Admin Service
```

Agora surge uma pergunta.

Como o usuário acessa isso?

---

## Primeira ideia

Criar quatro Services do tipo LoadBalancer.

```text
Internet

↓

LB Frontend
```

```text
Internet

↓

LB API
```

```text
Internet

↓

LB Auth
```

```text
Internet

↓

LB Admin
```

Funciona.

Mas...

- quatro IPs públicos;
- quatro Load Balancers;
- quatro certificados;
- quatro custos.

Não escala.

---

# O verdadeiro problema

HTTP já resolve isso há décadas.

Quando acessamos.

```text
https://api.google.com
```

ou

```text
https://mail.google.com
```

Tudo chega inicialmente ao mesmo ponto.

Depois é roteado.

O Kubernetes precisava de algo parecido.

---

# Modelo Mental

Imagine um shopping.

Existem centenas de lojas.

Mas existe apenas uma entrada principal.

O segurança pergunta.

> Para qual loja você deseja ir?

Depois encaminha o visitante.

Ingress faz exatamente isso.

---

# O que é um Ingress?

Esta é uma das definições mais importantes deste guia.

Um Ingress **não recebe requisições**.

Um Ingress **não faz proxy**.

Um Ingress **não faz balanceamento**.

Um Ingress é apenas um objeto da API contendo regras.

Exemplo.

```text
api.empresa.com

↓

Service API
```

```text
admin.empresa.com

↓

Service Admin
```

Nada mais.

---

# Quem realmente recebe o tráfego?

O Ingress Controller.

Ele normalmente é um Deployment.

Exemplos.

- NGINX Ingress Controller
- Traefik
- Kong
- HAProxy
- Envoy Gateway

Esse Deployment recebe as conexões externas.

---

# Arquitetura completa

Fluxo real.

```text
Internet

↓

Load Balancer

↓

Ingress Controller

↓

Ingress

↓

Service

↓

Pods
```

Observe.

O recurso Ingress está apenas fornecendo regras.

Quem executa essas regras é o Controller.

---

# Anatomia de um Ingress

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: api

spec:

  ingressClassName: nginx

  rules:

  - host: api.empresa.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: api-service

            port:

              number: 80
```

Observe.

Não existe nenhuma lógica de proxy aqui.

Apenas configuração.

---

# O papel do Ingress Controller

Imagine.

Ingress.

```text
api.empresa.com

↓

api-service
```

O Controller lê esse objeto.

Depois gera automaticamente sua configuração interna.

No caso do NGINX.

Gera um arquivo semelhante a.

```nginx
server {

    server_name api.empresa.com;

    location / {

        proxy_pass http://api-service;
    }

}
```

O mesmo acontece com Traefik, HAProxy e Kong.

---

# Fluxo completo de uma requisição

Imagine.

Usuário.

```text
https://api.empresa.com/users
```

Fluxo.

```text
Browser

↓

DNS

↓

Load Balancer

↓

Ingress Controller

↓

Ingress Rule

↓

Service

↓

EndpointSlice

↓

Pod

↓

Container
```

Esse fluxo ocorre em poucos milissegundos.

---

# Roteamento por Host

Ingress pode decidir baseado no domínio.

```text
api.empresa.com

↓

API
```

```text
admin.empresa.com

↓

Admin
```

```text
grafana.empresa.com

↓

Grafana
```

---

# Roteamento por Path

Também pode decidir pelo caminho.

```text
empresa.com/api

↓

API
```

```text
empresa.com/admin

↓

Admin
```

```text
empresa.com/docs

↓

Docs
```

---

# Um único IP público

Agora temos.

```text
Internet

↓

1 Load Balancer

↓

Ingress Controller

↓

20 aplicações
```

Esse é o principal benefício.

---

# TLS

Outro benefício.

HTTPS.

O certificado é configurado uma única vez.

```text
Internet

↓

HTTPS

↓

Ingress Controller

↓

HTTP interno
```

Ou.

```text
HTTPS

↓

HTTPS
```

Dependendo da arquitetura.

---

# TLS Termination

Muito comum.

```text
Cliente

↓

HTTPS

↓

Ingress Controller

↓

HTTP

↓

Service

↓

Pods
```

A descriptografia acontece no Controller.

---

# TLS Passthrough

Em alguns casos.

O Controller apenas encaminha.

```text
HTTPS

↓

Pod
```

A aplicação termina o TLS.

Menos comum.

---

# cert-manager

Gerenciar certificados manualmente é inviável.

O cert-manager automatiza.

```text
Ingress

↓

cert-manager

↓

Let's Encrypt

↓

Certificado

↓

Secret TLS
```

Renovação automática.

Sem intervenção humana.

---

# Exemplo com TLS

```yaml
tls:

- hosts:

  - api.empresa.com

  secretName: api-tls
```

O certificado fica armazenado em um Secret.

---

# IngressClass

Um cluster pode possuir vários Controllers.

Exemplo.

```text
NGINX
```

```text
Traefik
```

```text
Kong
```

Ingress informa qual deles utilizar.

```yaml
ingressClassName: nginx
```

---

# Casos reais

Empresa.

```text
empresa.com
```

Aplicações.

```text
api.empresa.com

admin.empresa.com

grafana.empresa.com

prometheus.empresa.com

argocd.empresa.com
```

Tudo compartilhando um único Ingress Controller.

---

# Hands-on

Criar Deployment.

```bash
kubectl apply -f deployment.yaml
```

Criar Service.

```bash
kubectl apply -f service.yaml
```

Criar Ingress.

```bash
kubectl apply -f ingress.yaml
```

Consultar.

```bash
kubectl get ingress
```

Descrever.

```bash
kubectl describe ingress
```

Verificar logs.

```bash
kubectl logs deployment/ingress-nginx-controller
```

---

# Limitações do Ingress

Ingress foi criado há muitos anos.

Hoje possui limitações.

- pouca padronização;
- funcionalidades específicas de cada Controller;
- excesso de annotations;
- recursos limitados para TCP e UDP.

---

# Gateway API

A comunidade Kubernetes está gradualmente migrando para a Gateway API.

Ela separa responsabilidades.

```text
Gateway

↓

HTTPRoute

↓

Backend
```

Arquitetura mais flexível.

Mais extensível.

Mais padronizada.

Nos próximos anos ela deverá substituir boa parte dos usos do Ingress.

---

# Comparando Services e Ingress

| Service | Ingress |
|----------|----------|
| Camada 4 (TCP/UDP) | Camada 7 (HTTP/HTTPS) |
| Expõe Pods | Roteia requisições HTTP |
| Balanceia conexões | Decide para qual Service enviar |
| Não entende URLs | Entende hosts e paths |

---

# Relação com os capítulos anteriores

Agora o fluxo completo de uma aplicação web é.

```text
Internet
    │
DNS
    │
Load Balancer
    │
Ingress Controller
    │
Ingress
    │
Service
    │
EndpointSlice
    │
Pods
    │
Containers
```

---

# Boas práticas

- Utilize apenas um ou poucos Ingress Controllers por cluster.
- Centralize TLS no Controller quando possível.
- Utilize cert-manager para certificados.
- Evite criar um LoadBalancer para cada aplicação.
- Planeje a migração para Gateway API em novos projetos.

---

# Erros comuns

> [!WARNING]

Achar que o recurso Ingress recebe tráfego.

Quem recebe é o Ingress Controller.

---

> [!WARNING]

Criar múltiplos Load Balancers desnecessariamente.

Um único Controller pode atender dezenas de aplicações.

---

> [!WARNING]

Misturar regras de aplicações sem organização.

Separe Ingresses por domínio, equipe ou contexto quando fizer sentido.

---

# Resumo

Neste capítulo aprendemos que:

- Ingress é apenas um objeto de configuração.
- O Ingress Controller é quem recebe as requisições.
- O Controller implementa proxy reverso e balanceamento HTTP.
- Um único Load Balancer pode atender diversas aplicações.
- TLS normalmente é terminado no Controller.
- Gateway API representa a evolução do modelo tradicional de Ingress.

---

# Checklist

Antes de continuar, verifique se você consegue responder:

- [ ] Qual a diferença entre Ingress e Ingress Controller?
- [ ] Por que um Service não substitui um Ingress?
- [ ] Como funciona o roteamento por Host?
- [ ] Como funciona o roteamento por Path?
- [ ] O que é TLS Termination?
- [ ] Qual o papel do cert-manager?
- [ ] Por que a Gateway API está ganhando espaço?

---

# Diagrama consolidado

```text
                    Internet
                        │
                       DNS
                        │
                 Load Balancer
                        │
              Ingress Controller
                        │
                    Ingress
                        │
                    Service
                        │
                 EndpointSlice
                        │
                 Deployment
                        │
                  ReplicaSet
                        │
                      Pods
              ┌─────────┼─────────┐
              │         │         │
        ConfigMap    Secret    Volumes
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
