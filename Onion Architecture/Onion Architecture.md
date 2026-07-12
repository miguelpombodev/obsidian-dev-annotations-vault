A **Onion Architecture** foi criada e proposta por **Jeffrey Palermo** em 2008 como uma resposta direta aos problemas da Arquitetura em Camadas (_Layered Architecture_).

Seu nome vem do formato visual semelhante ao de uma cebola: a arquitetura é composta por camadas concêntricas, onde o **núcleo** é a camada de **Domain** e a camada mais externa é a **Infrastructure**.

Sua regra fundamental é a mesma da **Clean Architecture**:

> **As dependências só podem apontar para dentro, nunca para fora.**

![[Pasted image 20260528121541.png]]

---

# Domain Layer

É a camada mais interna e mais protegida da arquitetura.

Aqui ficam:

- Entidades (_Entities_);
- [[Value Objects]];
- Enums de domínio;
- Exceções de domínio;
- Toda a lógica de negócio pura.

Nada nesta camada conhece banco de dados, APIs, frameworks, ORMs ou qualquer tecnologia externa.

```
// Entity
class Order {
  private _items: OrderItem[] = [];
  private _status: OrderStatus = OrderStatus.Draft;

  constructor(
    public readonly id: OrderId,
    public readonly customerId: CustomerId,
    private readonly createdAt: Date = new Date()
  ) {}

  addItem(item: OrderItem): void {
    if (this._status !== OrderStatus.Draft) {
      throw new DomainException('Só é possível adicionar itens em pedidos rascunho');
    }

    const existing = this._items.find(i => i.productId.equals(item.productId));

    if (existing) {
      existing.increaseQuantity(item.quantity);
    } else {
      this._items.push(item);
    }
  }

  confirm(): void {
    if (this._items.length === 0) {
      throw new DomainException('Pedido precisa ter ao menos um item');
    }

    this._status = OrderStatus.Confirmed;
  }

  get total(): Money {
    return this._items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero('BRL')
    );
  }

  get items(): ReadonlyArray<OrderItem> {
    return [...this._items];
  }
}
```

---

## Value Objects

Objetos de Valor representam conceitos do domínio que:

- Não possuem identidade;
- São imutáveis;
- São comparados pelos seus valores.

```
// Value Object — sem identidade, imutável e comparado por valor
class Money {
  constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {
    if (amount < 0) {
      throw new DomainException('Valor monetário não pode ser negativo');
    }
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new DomainException('Não é possível somar moedas diferentes');
    }

    return new Money(this.amount + other.amount, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount &&
           this.currency === other.currency;
  }

  static zero(currency: string): Money {
    return new Money(0, currency);
  }
}
```

---

## Enums do Domínio

Os enums que representam estados ou conceitos do negócio também pertencem à camada de domínio.

```
enum OrderStatus {
  Draft = 'draft',
  Confirmed = 'confirmed',
  Shipped = 'shipped',
  Cancelled = 'cancelled'
}
```

---

## O Modelo de Domínio

O **Domain Model** não importa absolutamente nada de fora.

Não há dependência de:

- Entity Framework;
- TypeORM;
- ASP.NET;
- NestJS;
- Express;
- RabbitMQ;
- PostgreSQL;
- APIs HTTP;
- Frameworks.

**Zero dependências externas.**

Isso torna o domínio totalmente independente da tecnologia utilizada.

---

# Domain Services

Nem toda regra de negócio pertence a uma única entidade.

Quando uma regra:

- envolve múltiplas entidades;
- necessita consultar dados externos;
- não faz sentido pertencer a uma entidade específica,

utilizamos um **Domain Service**.

É aqui que a Onion Architecture introduz uma característica muito importante:

> **As interfaces dos repositórios pertencem ao domínio, e não à infraestrutura.**

Ou seja:

- o domínio define o contrato;
- a infraestrutura implementa esse contrato.

Dessa forma, o domínio nunca depende da tecnologia de persistência.

---

## Interfaces dos Repositórios

```
// Interface do repositório — definida DENTRO do domínio.
// A infraestrutura implementará esta interface.
interface IOrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomer(customerId: CustomerId): Promise<Order[]>;
  save(order: Order): Promise<void>;
  nextId(): OrderId;
}

interface IProductRepository {
  findById(id: ProductId): Promise<Product | null>;
  findByIds(ids: ProductId[]): Promise<Product[]>;
}
```

Observe que não existe nenhuma referência a:

- TypeORM;
- Entity Framework;
- Prisma;
- MongoDB;
- PostgreSQL.

O domínio apenas define **o que precisa**, e não **como será implementado**.

---

## Domain Service

Um Domain Service encapsula regras que envolvem múltiplas entidades.

```
class OrderPricingService {

  // Calcula descontos considerando o histórico do cliente.
  // Essa responsabilidade não pertence nem a Order nem a Customer isoladamente.

  calculateDiscount(order: Order, customer: Customer): Money {

    const loyaltyYears = customer.loyaltyYearsAt(new Date());
    const orderTotal = order.total;

    if (loyaltyYears >= 5 && orderTotal.amount >= 500) {
      return new Money(orderTotal.amount * 0.15, orderTotal.currency);
    }

    if (loyaltyYears >= 2 && orderTotal.amount >= 200) {
      return new Money(orderTotal.amount * 0.08, orderTotal.currency);
    }

    return Money.zero(orderTotal.currency);
  }
}
```

Nesse exemplo, a regra depende simultaneamente de:

- Cliente;
- Pedido.

Como ela não pertence exclusivamente a nenhuma dessas entidades, ela é colocada em um Domain Service.

---

## Domain Service com Dependência de Interface

Um Domain Service pode depender de uma interface definida no próprio domínio.

```
class StockValidationService {

  constructor(
    private readonly stockRepo: IStockRepository
  ) {}

  async hasStock(
    productId: ProductId,
    quantity: number
  ): Promise<boolean> {

    const stock = await this.stockRepo.findByProduct(productId);

    return stock !== null &&
           stock.available >= quantity;
  }

  async reserveItems(items: OrderItem[]): Promise<void> {

    for (const item of items) {

      const hasEnough = await this.hasStock(
        item.productId,
        item.quantity
      );

      if (!hasEnough) {
        throw new DomainException(
          `Estoque insuficiente para o produto ${item.productId}`
        );
      }
    }
  }
}
```

Observe que esse serviço **não conhece** banco de dados algum.

Ele apenas utiliza uma interface (`IStockRepository`).

Quem implementará essa interface será a camada de **Infrastructure**, mantendo o domínio completamente desacoplado das tecnologias de persistência.

---

## Resumo da Domain Layer

A camada de domínio concentra tudo o que representa o negócio e suas regras. Ela deve permanecer completamente independente de qualquer tecnologia externa.

Em resumo, essa camada contém:

- **Entities**: representam os principais conceitos do negócio e encapsulam suas regras.
- **Value Objects**: objetos imutáveis, sem identidade, comparados por valor.
- **Enums**: estados e classificações do domínio.
- **Domain Exceptions**: exceções relacionadas às regras de negócio.
- **Repository Interfaces**: contratos que descrevem as operações necessárias para acessar os dados.
- **Domain Services**: regras que envolvem múltiplas entidades ou dependem de contratos definidos pelo próprio domínio.

O principal objetivo dessa camada é garantir que toda a lógica de negócio permaneça isolada da infraestrutura, permitindo que o domínio evolua sem depender de frameworks, bancos de dados ou qualquer tecnologia externa.
# Application Services

Esta é a camada que mais gera confusão para quem vem da **Clean Architecture**.

Na Onion Architecture, os **Application Services** exercem um papel semelhante aos **Use Cases** da Clean Architecture: eles orquestram o fluxo da aplicação.

Entretanto, existe uma diferença importante.

Além de coordenarem a execução do caso de uso, eles também **definem as interfaces dos serviços externos** que serão implementados pela camada de **Infrastructure**, como:

- Serviços de e-mail;
- Filas de mensagens;
- Armazenamento;
- Publicadores de eventos;
- APIs externas.

Em outras palavras:

> O domínio define os contratos para persistência (repositórios), enquanto a camada de Application define os contratos para serviços externos.

---

## Interfaces de serviços externos

```
// Interface de serviço externo — definida AQUI,
// implementada pela Infrastructure.

interface IEmailService {
  sendOrderConfirmation(to: string, order: Order): Promise<void>;
  sendShippingNotification(to: string, trackingCode: string): Promise<void>;
}

interface IEventPublisher {
  publish(event: DomainEvent): Promise<void>;
}
```

Da mesma forma que ocorre com os repositórios, a aplicação conhece apenas a **abstração**, nunca a implementação concreta.

---

## DTOs

Os **DTOs (Data Transfer Objects)** representam o contrato entre a aplicação e o mundo externo.

Um DTO de entrada representa os dados necessários para executar um caso de uso.

```
interface PlaceOrderDto {
  customerId: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
}
```

Já o DTO de saída representa aquilo que será retornado pela aplicação.

```
interface OrderResultDto {
  orderId: string;
  total: number;
  currency: string;
  status: string;
}
```

Os DTOs impedem que entidades do domínio sejam expostas diretamente para outras camadas.

---

## Application Service

O Application Service é responsável por **orquestrar** o fluxo da aplicação.

Ele **não contém regras de negócio**.

Sua responsabilidade é coordenar os objetos corretos na ordem correta.

```
class OrderApplicationService {
  constructor(
    private readonly orderRepo: IOrderRepository,
    private readonly productRepo: IProductRepository,
    private readonly stockService: StockValidationService,
    private readonly pricingService: OrderPricingService,
    private readonly emailService: IEmailService,
    private readonly eventPublisher: IEventPublisher
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<OrderResultDto> {
    const customerId = new CustomerId(dto.customerId);

    const customer = await this.customerRepo.findById(customerId);

    if (!customer)
      throw new ApplicationException("Cliente não encontrado");

    const order = new Order(
      this.orderRepo.nextId(),
      customerId
    );

    for (const itemDto of dto.items) {

      const productId = new ProductId(itemDto.productId);

      const product = await this.productRepo.findById(productId);

      if (!product)
        throw new ApplicationException(
          `Produto ${itemDto.productId} não encontrado`
        );

      await this.stockService.hasStock(
        productId,
        itemDto.quantity
      );

      order.addItem(
        new OrderItem(
          productId,
          itemDto.quantity,
          product.price
        )
      );
    }

    const discount =
      this.pricingService.calculateDiscount(order, customer);

    order.applyDiscount(discount);

    order.confirm();

    await this.orderRepo.save(order);

    await this.emailService.sendOrderConfirmation(
      customer.email,
      order
    );

    await this.eventPublisher.publish(
      new OrderPlacedEvent(order.id)
    );

    return {
      orderId: order.id.value,
      total: order.total.amount,
      currency: order.total.currency,
      status: order.status
    };
  }
}
```

Observe a responsabilidade desse serviço.

Ele:

- cria objetos;
- consulta repositórios;
- chama Domain Services;
- chama entidades;
- persiste dados;
- envia notificações;
- publica eventos.

Mas **não toma decisões de negócio**.

Quem decide é o domínio.

O Application Service apenas coordena a execução dessas decisões.

---

# Infrastructure

Esta é a camada mais externa da Onion Architecture.

Aqui ficam todas as implementações concretas das interfaces definidas nas camadas internas.

Alguns exemplos:

- Banco de dados;
- ORM;
- RabbitMQ;
- Kafka;
- Redis;
- APIs externas;
- Serviços de e-mail;
- Controllers HTTP;
- Frameworks.

Toda tecnologia específica pertence a esta camada.

---

## Implementação de Repositórios

```
@Injectable()
class TypeOrmOrderRepository implements IOrderRepository {

  constructor(
    @InjectRepository(OrderEntity)
    private readonly repo: Repository<OrderEntity>
  ) {}

  async findById(id: OrderId): Promise<Order | null> {

    const entity = await this.repo.findOne({
      where: { id: id.value },
      relations: ['items']
    });

    if (!entity)
      return null;

    return OrderMapper.toDomain(entity);
  }

  async save(order: Order): Promise<void> {

    const entity =
      OrderMapper.toPersistence(order);

    await this.repo.save(entity);
  }

  nextId(): OrderId {
    return new OrderId(uuid());
  }
}
```

Perceba que agora aparecem detalhes específicos da tecnologia utilizada:

- TypeORM;
- Repository;
- Decorators;
- Injeção de dependência.

Tudo isso permanece isolado da camada de domínio.

---

## Mappers

Os mappers fazem a tradução entre:

- Modelo de domínio;
- Modelo de persistência.

```
class OrderMapper {

  static toDomain(entity: OrderEntity): Order {

    return Order.reconstitute(
      new OrderId(entity.id),
      new CustomerId(entity.customerId),
      entity.status as OrderStatus,
      entity.items.map(i =>
        new OrderItem(
          new ProductId(i.productId),
          i.quantity,
          new Money(i.unitPrice, i.currency)
        )
      ),
      entity.createdAt
    );
  }

  static toPersistence(order: Order): OrderEntity {

    const entity = new OrderEntity();

    entity.id = order.id.value;

    entity.customerId = order.customerId.value;

    entity.status = order.status;

    entity.items = order.items.map(i => {

      const itemEntity = new OrderItemEntity();

      itemEntity.productId = i.productId.value;

      itemEntity.quantity = i.quantity;

      itemEntity.unitPrice =
        i.subtotal.amount / i.quantity;

      return itemEntity;
    });

    return entity;
  }
}
```

Esse é um detalhe importante da Onion Architecture.

As entidades do domínio nunca são entidades do ORM.

Elas são convertidas através de mappers.

---

## Controllers

Na Onion Architecture, Controllers também fazem parte da camada de Infrastructure.

```
@Controller("/orders")
class OrderController {

  constructor(
    private readonly orderService: OrderApplicationService
  ) {}

  @Post()

  async create(
    @Body() body: CreateOrderRequest,
    @Res() res: Response
  ) {

    const result =
      await this.orderService.placeOrder({

        customerId: body.customer_id,

        items: body.items.map(i => ({
          productId: i.product_id,
          quantity: i.qty
        }))
      });

    return res.status(201).json({

      order_id: result.orderId,

      total_amount: result.total,

      currency: result.currency
    });
  }
}
```

O Controller apenas traduz HTTP para o modelo utilizado pela aplicação.

Nenhuma regra de negócio deve existir aqui.

---

## Serviços Externos

Outro exemplo são implementações de serviços externos.

```
class SendGridEmailService
implements IEmailService {

  constructor(
    private readonly client: SendGridClient
  ) {}

  async sendOrderConfirmation(
    to: string,
    order: Order
  ): Promise<void> {

    await this.client.send({

      to,

      subject: `Pedido #${order.id.value} confirmado`,

      html: this.buildConfirmationHtml(order)
    });
  }
}
```

Observe novamente que o domínio não conhece SendGrid.

Ele conhece apenas a interface `IEmailService`.

---

# Estrutura de Pastas na Onion Architecture

```
src/
├── domain/
│   ├── model/
│   │   ├── order/
│   │   │   ├── Order.ts
│   │   │   ├── OrderItem.ts
│   │   │   └── OrderStatus.ts
│   │   └── shared/
│   │       ├── Money.ts
│   │       └── DomainException.ts
│   │
│   └── services/
│       ├── OrderPricingService.ts
│       ├── StockValidationService.ts
│       └── repositories/
│           ├── IOrderRepository.ts
│           └── IProductRepository.ts
│
├── application/
│   ├── services/
│   │   ├── OrderApplicationService.ts
│   │   └── CustomerApplicationService.ts
│   ├── dtos/
│   │   ├── PlaceOrderDto.ts
│   │   └── OrderResultDto.ts
│   └── interfaces/
│       ├── IEmailService.ts
│       └── IEventPublisher.ts
│
└── infrastructure/
    ├── persistence/
    │   ├── TypeOrmOrderRepository.ts
    │   ├── entities/
    │   └── mappers/
    ├── email/
    ├── http/
    └── container.ts
```

Essa organização evidencia a principal regra da Onion Architecture: as dependências sempre apontam para o centro da aplicação.

---

# O que a Onion Architecture resolve e onde ela para

O maior avanço da Onion Architecture em relação à Arquitetura em Camadas é a **inversão completa das dependências**.

O domínio deixa de depender da infraestrutura.

Em vez disso, é a infraestrutura que depende do domínio.

Isso é possível porque as interfaces dos repositórios são definidas dentro do próprio domínio e implementadas externamente.

Entretanto, quando comparada à Clean Architecture, a Onion Architecture é propositalmente menos prescritiva em alguns aspectos.

O primeiro é que ela não separa formalmente:

- **o que a aplicação faz** (_Use Cases_);
- **como ela se comunica com o mundo externo** (_Interface Adapters_).

Ambas as responsabilidades coexistem dentro dos **Application Services**.

O segundo é que elementos como:

- Controllers;
- Presenters;
- Repositórios;
- Adaptadores de infraestrutura;

ficam agrupados dentro do mesmo anel (**Infrastructure**), sem uma camada explícita de adaptação entre eles.

Na prática, isso significa que equipes que utilizam Onion Architecture costumam concentrar parte da lógica de adaptação diretamente nos **Application Services** ou nos **Controllers**.

Essa abordagem funciona muito bem para projetos menores e médios, pois mantém a arquitetura simples e objetiva.

Entretanto, conforme o sistema cresce, a separação entre **orquestração de negócio** e **adaptação de formatos** tende a ficar menos evidente, aumentando a complexidade de manutenção. É justamente esse ponto que a **Clean Architecture** procura resolver ao introduzir camadas específicas para **Use Cases**, **Interface Adapters** e **Frameworks & Drivers**, tornando as responsabilidades mais explícitas e facilitando a evolução de sistemas de grande porte.