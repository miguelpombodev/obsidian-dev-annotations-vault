
## Providers

[[AWS]]
[[Azure]]

---

Computação na Nuvem é um modelo no qual podemos acessar recursos computacionais, como armazenamento e processamento, pela internet sob demanda e de forma flexível.

![[Cloud/Attachments/image.png]]

Os benefícios são:

- Redução de custos;
- Escalabilidade flexível;
- Acesso global

Os modelos de nuvem são:

- Pública (AWS, Azure, GCP, etc)
- Híbrida;
- Privada

### Serviços no Provedor

Um provedor de computação na nuvem tem diversos serviços dos quais utilizamos frequentemente, como:

- Autenticação e autorização de usuários;
- Database;
- Servidores Web e VMs;
- Docker Registries;
- Kubernetes e outros orquestradores

![[Cloud/Attachments/image 1.png]]

## Por que usar a nuvem

A computação em nuvem nos dar muito poder, pois conseguimos subir um servidor web em minutos, conseguimos aumentar ou diminuir dimensionamento dos recursos em um clique, e escalar nossas instâncias.

Além de termos o modelo pay-per-use, ou seja, pagando apenas pelo o que usa. Por exemplo, se subirmos uma máquina virtual e depois desistirmos de seu uso, podemos excluí-la e seremos cobrandos apenas pelo tempo em que ela esteve de pé.

![[Cloud/Attachments/image 2.png]]

## Modelos de Serviços na Nuvem

Atualmente, temos 3 modelos principais de serviços em cloud:

- Iaas (Infrastructure as a Service);
- PaaS (Platform as a Service);
- Saas (Software as a Service)

![[Cloud/Attachments/image 3.png]]

### SaaS 🧑🏽‍💻

Saas significa “Software As A Service”, e é onde a distribuição de software é feita via plataforma web, é uma forma de hospedar e distribuir centralmente para os clientes via internet. 

As aplicações web são executadas nos servidores dos provedores SaaS. 

Essa aplicações podem ser usadas no computador, tablet ou telefones, de maneira simples e remota. Por exemplo, **Salesforce**,

### IaaS :crane:

IaaS significa “Infrastructure As A Service”, e é onde permite que clientes aluguem um espaço em datacenter virtual oferecido por um servidor de IaaS, como a **Azure**, a **AWS**, o **GCP**, etc.

Ao invés de comprar um servidor on-premise (local), o cliente aluga os recursos computacionais, assim ele mesmo pode configurar e gerir o próprio sistema operacional e suas aplicações.

### PaaS 🧩

PaaS significa “Plataform As A Service”, nesse podemos dizer que é o meio termo entre SaaS e IaaS, e é onde é disponibilizada uma plataforma de desenvolvimento, testes e implantações de aplicações de forma simplificada sem a necessidade de configurar uma infrastrutura.

Segue uma ilustração desses modelos e o que eles abrangem:

![[Cloud/Attachments/image 4.png]]

![[image 5.png]]

![[image 6.png]]

### Outros modelos

Temos também outros modelos além desses mostrados, como **FaaS** (Function As A Service), **BaaS** (Backend As A Service), **DaaS** (Database As A Service) e **Serverless**.

![[image 7.png]]

## Tipos de Escalonamento

**Escalonamento Vertical e Horizontal**

Resumindo:

**Vertical** → Cresce a capacidade da máquina (Scale Up)

**Horizontal** → Cresce a quantidade de máquinas (Scale Out)

O mais comum é criarmos maquinas mais básicas e ir crescendo o número de máquinas.