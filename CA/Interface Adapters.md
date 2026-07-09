-  In this layer, we convert the data to the most convenient format for a Use Case or even the most convenient format for external world;
-  Here we have Controllers, Presenters and Gateways/ Repositories implementations

```ts
// Controller — recebe HTTP, chama o Use Case
class OrderController {
  constructor(private placeOrderUseCase: PlaceOrderUseCase) {}

  async handlePost(req: HttpRequest, res: HttpResponse): Promise<void> {
    try {
      // Converte o mundo HTTP para o formato do Use Case
      const input: PlaceOrderInput = {
        customerId: req.body.customer_id,
        customerEmail: req.body.email,
        items: req.body.items.map((i: any) => ({
          productId: i.product_id,
          quantity: i.qty
        }))
      };

      const output = await this.placeOrderUseCase.execute(input);

      // Converte a resposta do Use Case para o formato HTTP
      res.status(201).json({
        order_id: output.orderId,
        total_amount: output.total
      });
    } catch (err) {
      res.status(400).json({ error: err.message });
    }
  }
}

// Repository concreto — implementa a interface definida no Use Case
class PostgresOrderRepository implements OrderRepository {
  constructor(private db: DatabaseConnection) {}

  async save(order: Order): Promise<void> {
    // Converte a entidade de domínio para a estrutura do banco
    await this.db.query(
      `INSERT INTO orders (id, customer_id, total, status)
       VALUES ($1, $2, $3, $4)`,
      [order.id, order.customerId, order.getTotal(), order.status]
    );
  }

  async findById(id: string): Promise<Order | null> {
    const row = await this.db.query('SELECT * FROM orders WHERE id = $1', [id]);
    if (!row) return null;
    // Converte a linha do banco de volta para uma entidade de domínio
    return Order.reconstitute(row);
  }
}
```

Note that **PostgresOrderRepository** implements **OrderRepository** (defined in the Use Case), and not the other way around. The dependency arrow goes from the outside in.