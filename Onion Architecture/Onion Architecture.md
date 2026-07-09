The Onion Architecture was designed and proposed by Jeffrey Palermo in 2008 as direct reaction to the problems of Layered Architecture. 
Its name comes from the visual form, like an onion, we have layers, where the core is the Domain layer and the Infrastructure layer is most outside layer at the architecture.
The fundamental rule is the same to the  Clean Architecture: *==dependency can only point inward, never outward.==*

![[Pasted image 20260528121541.png]]


### Domain Layer
The most inner and most protected layer, here we have the entities and the [[Value Objects]], the fundamental business concepts without any external dependency.
```ts
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

```ts
// Value Object — sem identidade, imutável, comparado por valor
class Money {
  constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {
    if (amount < 0) throw new DomainException('Valor monetário não pode ser negativo');
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new DomainException('Não é possível somar moedas diferentes');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  static zero(currency: string): Money {
    return new Money(0, currency);
  }
}
```
```ts
// Enum de domínio — também mora aqui
enum OrderStatus {
  Draft = 'draft',
  Confirmed = 'confirmed',
  Shipped = 'shipped',
  Cancelled = 'cancelled'
}
```

The Domain Model does not import a thing. **==Zero external dependencies.==** 

### Domain Services
When a business rule involves multiples entities or requires an external data access to work, it does not fit in a single entity, for this we use Domain Services.
 Here, the Onion Architecture uses something important, that is the repositories interface as defined in this layer, and not in the Infrastructure layer.
 The domain creates the contract, and the infrastructure implements it.
 
 ```ts
 // Interface do repositório — definida DENTRO do domínio
// A infraestrutura vai implementar isso, não o contrário
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

```ts
// Domain Service — lógica que abrange múltiplas entidades
class OrderPricingService {
  // Calcula desconto considerando o histórico do cliente
  // Isso não cabe em Order nem em Customer isoladamente
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

```ts
// Domain Service com dependência de interface (não de implementação)
class StockValidationService {
  constructor(private readonly stockRepo: IStockRepository) {}

  async hasStock(productId: ProductId, quantity: number): Promise<boolean> {
    const stock = await this.stockRepo.findByProduct(productId);
    return stock !== null && stock.available >= quantity;
  }

  async reserveItems(items: OrderItem[]): Promise<void> {
    for (const item of items) {
      const hasEnough = await this.hasStock(item.productId, item.quantity);
      if (!hasEnough) {
        throw new DomainException(`Estoque insuficiente para produto ${item.productId}`);
      }
    }
  }
}
```

### Application Services
This is the layer that most confuses those coming from Clean Architecture. Onion's Application Services do what CA's Use Cases do, but with a subtle difference: they also define the interfaces that the Infrastructure layer will implement for external services (email, queues, storage), in addition to orchestrating the flow.
```ts
// Interface de serviço externo — definida AQUI, implementada na Infrastructure
interface IEmailService {
  sendOrderConfirmation(to: string, order: Order): Promise<void>;
  sendShippingNotification(to: string, trackingCode: string): Promise<void>;
}

interface IEventPublisher {
  publish(event: DomainEvent): Promise<void>;
}
```

```ts
// DTO de entrada — define o contrato com o mundo externo
interface PlaceOrderDto {
  customerId: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
}

// DTO de saída — define o que retorna para a camada de Infrastructure
interface OrderResultDto {
  orderId: string;
  total: number;
  currency: string;
  status: string;
}
```

```ts
// Application Service — orquestra domínio, não contém regras de negócio
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
    if (!customer) throw new ApplicationException('Cliente não encontrado');

    const order = new Order(this.orderRepo.nextId(), customerId);

    for (const itemDto of dto.items) {
      const productId = new ProductId(itemDto.productId);
      const product = await this.productRepo.findById(productId);
      if (!product) throw new ApplicationException(`Produto ${itemDto.productId} não encontrado`);

      await this.stockService.hasStock(productId, itemDto.quantity);
      order.addItem(new OrderItem(productId, itemDto.quantity, product.price));
    }

    const discount = this.pricingService.calculateDiscount(order, customer);
    order.applyDiscount(discount);
    order.confirm();

    await this.orderRepo.save(order);
    await this.emailService.sendOrderConfirmation(customer.email, order);
    await this.eventPublisher.publish(new OrderPlacedEvent(order.id));

    return {
      orderId: order.id.value,
      total: order.total.amount,
      currency: order.total.currency,
      status: order.status
    };
  }
}
```

The Application Service coordinates, however it does not decides. It calls the Domain Service to calculate, calls the entity to apply the rules, and then it persists and notifies. No business logic here.

### Infrastructure
The most external layer. Here stay the concrete implementations of all interfaces defined at the inner layers, such as databases, queues, storages, HTTP frameworks, email services, ORM.
```ts
// Implementação do repositório — TypeORM como detalhe de infraestrutura
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
    if (!entity) return null;
    return OrderMapper.toDomain(entity); // converte ORM entity → Domain entity
  }

  async save(order: Order): Promise<void> {
    const entity = OrderMapper.toPersistence(order); // converte Domain → ORM
    await this.repo.save(entity);
  }

  nextId(): OrderId {
    return new OrderId(uuid());
  }
}
```

```ts
// Mapper — traduz entre representação de domínio e de persistência
class OrderMapper {
  static toDomain(entity: OrderEntity): Order {
    const order = Order.reconstitute(
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
    return order;
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
      itemEntity.unitPrice = i.subtotal.amount / i.quantity;
      return itemEntity;
    });
    return entity;
  }
}
```

```ts
// Controller HTTP — também é Infrastructure na Onion
@Controller('/orders')
class OrderController {
  constructor(private readonly orderService: OrderApplicationService) {}

  @Post()
  async create(@Body() body: CreateOrderRequest, @Res() res: Response) {
    const result = await this.orderService.placeOrder({
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

```ts
// Implementação do serviço de email
class SendGridEmailService implements IEmailService {
  constructor(private readonly client: SendGridClient) {}

  async sendOrderConfirmation(to: string, order: Order): Promise<void> {
    await this.client.send({
      to,
      subject: `Pedido #${order.id.value} confirmado`,
      html: this.buildConfirmationHtml(order)
    });
  }
}
```

### Folder Structure at Onion Architecture
```bash
src/
├── domain/
│   ├── model/                    ← Anel 1: Domain Model
│   │   ├── order/
│   │   │   ├── Order.ts          ← Entidade
│   │   │   ├── OrderItem.ts
│   │   │   └── OrderStatus.ts
│   │   └── shared/
│   │       ├── Money.ts          ← Value Object
│   │       └── DomainException.ts
│   │
│   └── services/                 ← Anel 2: Domain Services
│       ├── OrderPricingService.ts
│       ├── StockValidationService.ts
│       └── repositories/         ← Interfaces de repo definidas AQUI
│           ├── IOrderRepository.ts
│           └── IProductRepository.ts
│
├── application/                  ← Anel 3: Application Services
│   ├── services/
│   │   ├── OrderApplicationService.ts
│   │   └── CustomerApplicationService.ts
│   ├── dtos/
│   │   ├── PlaceOrderDto.ts
│   │   └── OrderResultDto.ts
│   └── interfaces/               ← Interfaces de serviços externos
│       ├── IEmailService.ts
│       └── IEventPublisher.ts
│
└── infrastructure/               ← Anel 4: Infrastructure
    ├── persistence/
    │   ├── TypeOrmOrderRepository.ts
    │   ├── entities/
    │   │   └── OrderEntity.ts
    │   └── mappers/
    │       └── OrderMapper.ts
    ├── email/
    │   └── SendGridEmailService.ts
    ├── http/
    │   └── OrderController.ts
    └── container.ts              ← Composição e injeção de dependências
```

### What Onion Solves and Where It Stops
Onion's major advancement over Layered Architecture is the complete inversion of dependencies: the domain never knows the infrastructure. This is guaranteed by the rule that repository interfaces reside within the domain.

However, compared to Clean Architecture, Onion is deliberately less prescriptive in two ways. The first is that it doesn't formally separate "what the app does" (Use Cases) from "how the app connects" (Interface Adapters) — all of this coexists within Application Services. The second is that concrete Controllers, Presenters, and Repositories all reside within the same Infrastructure ring, without an explicit translation layer between them.

In practice, this means that teams using Onion tend to place adaptation logic directly in Application Services or Controllers, and the distinction between "business orchestration" and "format translation" becomes implicit — which works well in smaller projects but can grow in complexity as the system scales.