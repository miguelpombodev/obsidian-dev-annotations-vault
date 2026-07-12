Antes de qualquer linha de código, precisamos entender o problema. Sem isso, esse padrão parece apenas uma burocracia desnecessária.

---
## O problema real

Imagine o seguinte fluxo em um **Catalog Service**:

```
1. Produto criado → salvo no PostgreSQL  ✅
2. Publica evento no RabbitMQ          💥 falha aqui
```

O banco de dados contém o produto, porém o RabbitMQ não recebeu o evento. O **Notification Service** nunca ficará sabendo que um produto foi criado.

Para o usuário, nenhum erro é percebido — foi retornado um **HTTP 201 Created**.

Essa é uma **inconsistência silenciosa** — o pior tipo de problema em sistemas distribuídos, porque você só descobre que algo está errado quando um cliente reclama.

A tentação natural é inverter a ordem:

```
1. Publica evento no RabbitMQ  ✅
2. Produto criado → salvo no PostgreSQL  💥 falha aqui
```

Agora o evento foi publicado, mas o produto não existe no banco de dados. Outros serviços irão reagir à criação de um produto que, na realidade, nunca foi persistido.

Não existe uma ordem segura para executar duas operações distribuídas sem algum tipo de coordenação. Esse é um dos problemas fundamentais dos sistemas distribuídos.

---

## Por que o Two-Phase Commit (2PC) não é a resposta?

A solução teórica seria utilizar um protocolo de commit distribuído (**2PC**).

O problema prático é que:

- O RabbitMQ não suporta 2PC.
- Mesmo entre tecnologias que suportam (como alguns bancos de dados), o protocolo é lento, complexo e, caso o coordenador falhe, todos os participantes podem ficar bloqueados indefinidamente.

Em arquiteturas modernas de microsserviços, o **2PC é considerado um antipadrão**, pois sacrifica disponibilidade em troca de consistência justamente em cenários onde ambos são importantes.

---

## A ideia central do Outbox Pattern

A sacada do Outbox Pattern é extremamente elegante:

Você garante atomicidade **apenas dentro de uma única transação local**.

Se os dados da aplicação e a mensagem estão no mesmo banco de dados, ambos podem ser persistidos de forma atômica.

Depois, um processo separado lê essas mensagens e as publica no RabbitMQ.

```
Transação local (atômica):

├── INSERT products          (os dados)
└── INSERT outbox_messages   (o evento serializado)

Background Worker (processo separado):

├── SELECT outbox_messages WHERE processed_at IS NULL
├── Publica no RabbitMQ
└── UPDATE outbox_messages SET processed_at = now()
```

Ou o banco confirma ambos os **INSERTs**, ou ambos falham.

Nunca haverá um sem o outro.

O worker pode falhar, reiniciar e tentar novamente quantas vezes forem necessárias — o evento continuará armazenado no banco aguardando processamento.

---

## As consequências que você precisa aceitar

O Outbox Pattern fornece garantia de **at-least-once delivery** (_entrega pelo menos uma vez_).

Isso significa que um evento pode ser publicado mais de uma vez.

Por quê?

O worker primeiro publica o evento no RabbitMQ e somente depois atualiza o campo **processed_at**.

Caso ele falhe entre essas duas operações, na próxima execução encontrará novamente a mensagem como pendente e fará uma nova publicação.

Isso **não é um bug do Outbox Pattern**.

É uma consequência inevitável dos sistemas distribuídos.

A resposta correta não é tentar implementar **exactly-once delivery** (praticamente inviável sem 2PC), mas sim tornar os consumidores **idempotentes**.

É justamente por isso que em [[Domain Events]] existe o campo **EventId**.

O consumidor mantém um registro dos eventos já processados e ignora duplicatas.

```
Consumidor recebe um evento com EventId = "abc-123"

├── Já processei "abc-123"?
│   └── Sim → ignora
└── Não
    └── Processa e registra "abc-123"
```

---

## Campos da classe OutboxMessage e por que eles existem

```C#
public sealed class OutboxMessage
{
    public Guid Id { get; }                    // idempotência no consumidor
    public string Type { get; }                // classe usada para desserializar o Payload
    public string Payload { get; }             // evento serializado em JSON
    public DateTimeOffset OccurredOn { get; }  // momento em que o evento ocorreu no domínio
    public DateTimeOffset? ProcessedOn { get; } // null = pendente, valor = já publicado
}
```

### **Id**

Cada mensagem possui sua própria identidade.

O consumidor utiliza esse identificador para detectar e ignorar mensagens duplicadas.

---

### **Type**

Armazena o nome completo da classe do evento.

Exemplo:

```
Catalog.Domain.Events.ProductCreatedEvent
```

Sem essa informação, ao ler o JSON do banco de dados você não saberia para qual tipo desserializar o conteúdo.

---

### **Payload**

Contém o evento completo serializado em JSON.

É totalmente independente da tecnologia utilizada.

Se amanhã você trocar RabbitMQ por Kafka, por exemplo, o payload continuará exatamente o mesmo.

---

### **OccurredOn**

O timestamp realmente importante é o momento em que o evento aconteceu no domínio, e não quando foi publicado.

Esse campo é utilizado para auditoria, ordenação de eventos e análises temporais.

---

### **ProcessedOn** (nullable)

É o indicador de estado da mensagem.

- **NULL** → mensagem pendente.
- **Possui valor** → mensagem já publicada.

O worker utiliza exatamente esse campo como critério de busca:

```
WHERE processed_on IS NULL
```

É uma solução simples, eficiente e facilmente indexável.

---

## Fluxo completo dentro de um contexto

```
HTTP POST /products
│
├── ProductService.CreateAsync()
│   ├── var product = Product.Create(...)
│   │   └── product.AddDomainEvent(new ProductCreatedEvent(product.Id))
│   │
│   └── await _unitOfWork.CommitAsync()
│       ├── Detecta os Domain Events pendentes da entidade
│       ├── Serializa cada evento em OutboxMessage
│       ├── INSERT INTO products         ┐
│       ├── INSERT INTO outbox_messages  ┘ mesma transação
│       └── COMMIT
│
└── Retorna HTTP 201 Created

[Background Worker — processo separado]

│
├── SELECT * FROM outbox_messages WHERE processed_on IS NULL
├── Para cada mensagem:
│   ├── Desserializa o Payload utilizando o tipo armazenado em Type
│   ├── Publica no RabbitMQ
│   └── UPDATE outbox_messages SET processed_on = now() WHERE id = ?
└── Aguarda N segundos e repete o processo
```