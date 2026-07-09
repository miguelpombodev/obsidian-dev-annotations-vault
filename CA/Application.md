-  This layer orchestrates the application flow through **Use Cases**;
-  Each **Use Case** represent an action that the user may do such as "Create an user", "Update its data","Delete its cart";
-  Works along with the entities defined in [[Domain]] layer and defines the operation workflow;
-  The layer does not know how the data is coming from (HTTP, gRPC, CLI) or how it's being stored (PostgreSQL, MongoDB).
-  The external world communication is done through interfaces (ports), it's never done by these interface's implementation.

```typescript
// Interface (port)
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

interface EmailService {
  sendConfirmation(to: string, orderId: string): Promise<void>;
}

// Use Case
class PlaceOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,  // injeção de interface
    private emailService: EmailService,
    private productRepository: ProductRepository
  ) {}

  async execute(input: PlaceOrderInput): Promise<PlaceOrderOutput> {
    const order = new Order(input.customerId);

    for (const item of input.items) {
      const product = await this.productRepository.findById(item.productId);
      if (!product) throw new Error(`Produto ${item.productId} não encontrado`);
      order.addItem(product, item.quantity);
    }

    order.confirm();

    await this.orderRepository.save(order);
    await this.emailService.sendConfirmation(input.customerEmail, order.id);

    return { orderId: order.id, total: order.getTotal() };
  }
}
```

