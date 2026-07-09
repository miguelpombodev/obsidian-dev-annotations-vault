Before any piece of code, let's feel the ground. Without it, this pattern feels like unnecessary bureaucracy.

---
## Real Problem
Picture the following workflow at a Catalog Service
```
1. Created product → stores at PostgreSQL  ✅
2. Publish event at RabbitMQ            💥 fails here
```

The database has the product, however RabbitMQ does not have the event. The Notification Service never knew that a product has been created. No visible error for the user — A **201 Created Status Code** was returned.

This is a silence inconsistency — The worst kind of bug in distributed systems, because you don't realize that you have a problem until the client complains about it.

The natural temptation is invert this order, such as:
```
1. Publish event at RabbitMQ  ✅
2. Created product → stores at PostgreSQL         💥 fails here
```

Now the event was published but the product doesn't exist in database. Others services will react about a product that doesn't exist.
There is no safe order for two operations in distributed systems without coordination. This is the distributed systems's fundamental issue.

---
## Why Two-Phase Commit is not the answer?

The theoretical solution would be a distributed commit protocol (**2PC**). The practical problem:
RabbitMQ does not support 2PC. Even those that do (some databases), the protocol is slow, complex and a failed coordinator locks all participants indefinitely. 
In modern microservices, 2PC is considered and anti-pattern, it trades availability for consistency at a point that needs both.

---
## Outbox's central idea
the insight is elegant: you only guarantee the atomicity inside an unique local transaction.

If the database and message are in the same database, you can commit them both atomically. A separate process reads the database and publishes them to RabbitMQ independently.
```
Transação local (atômica):
  ├── INSERT products          (o dado)
  └── INSERT outbox_messages   (o evento serializado)

Background Worker (separado):
  ├── SELECT outbox_messages WHERE processed_at IS NULL
  ├── Publica no RabbitMQ
  └── UPDATE outbox_messages SET processed_at = now()
```

Or the database commits both **INSERTs** or both fail. Never one without the other, The worker can fail, restart, try again — the event will always be in the database waiting.

---
## Consequences that you must accept
The Outbox Pattern bring the **at-least-once delivery** — the event can be published more than once. And why?

The worker publishes in RabbitMQ and then update **processed_at** field. If it fails between both steps, at the next execution it will find the message as unprocessed and publish it again.

This is not a bug in Outbox, it is a fundamental consequence in distributed systems. The right answer is not to try to guarantee exactly-once (unfeasible without 2PCs), but to make the consumers idempotent.

And that's why that in [[Domain Events]] we have the field EventID. A consumer keeps the IDs already processed and ignore the duplicates:
```
Consumer receive the event with EventId = "abc-123"
├── Have I already processed "abc-123"? → ignore
└── I have not processed it → process and register "abc-123"
```

---
OutboxMessage class fields and whys?
```cs
public sealed class OutboxMessage { 
	public Guid Id { get; } // idempotência no consumidor 
	public string Type { get; } // qual classe desserializar o Payload 
	public string Payload { get; } // evento serializado em JSON 
	public DateTimeOffset OccurredOn { get; } // quando o evento aconteceu no domínio 
	public DateTimeOffset? ProcessedOn { get; } // null = pendente, valor = já publicado 
}
```

**Id**: Each message has its own identity. The consumer uses this for deduplication

**Type**: The event class's complete name (**e.g Catalog.Domain.Events.ProductCreatedEvent**). Without this, when reading from the database you get a JSON but you don't know which type to deserialize it into

**Payload**: The complete event in JSON. Technology agnostic, if you switch RabbitMQ for Kafka, the payload never changes.

**OccuredOn**: The relevant timestamp is when the event occurred in the domain, not when it was processed. For auditing and ordering events, the timestamp is used.

**ProcessedOn**(nullable): The state discriminator. NULL is worker's search criterion, WHERE processed_on IS NULL. Simple, indexable, efficient.

---
## The complete workflow in a context
```
HTTP POST /products
│
├── ProductService.CreateAsync()
│   ├── var product = Product.Create(...)
│   │   └── product.AddDomainEvent(new ProductCreatedEvent(product.Id))
│   │
│   └── await _unitOfWork.CommitAsync()
│       ├── Detecta domain events pendentes na entidade
│       ├── Serializa cada evento em OutboxMessage
│       ├── INSERT INTO products         ┐
│       ├── INSERT INTO outbox_messages  ┘ mesma transação
│       └── COMMIT
│
└── Retorna 201 Created

[background worker — processo separado]
│
├── SELECT * FROM outbox_messages WHERE processed_on IS NULL
├── Para cada mensagem:
│   ├── Desserializa Payload para o tipo em Type
│   ├── Publica no RabbitMQ
│   └── UPDATE outbox_messages SET processed_on = now() WHERE id = ?
└── Aguarda N segundos e repete
```
