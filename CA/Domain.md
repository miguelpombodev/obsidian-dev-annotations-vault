-  More internal, more reliable and stabler layer;
-  It's where we have the company's business rules, these are entities around which the business rules revolve. In short, this layer encompasses the rules and entities that would exist even if the software did not exist;
-  These rules does not change because of a framework or technical decision;
-  Here the entities do not concern about any technical tool, such as databases, HTTP, framework, etc. ==**WE ONLY HAVE BUSINESS RULES!**==.

A example of a business rule or entity is:
- "An **Order** needs to have a total";
- "An **User** always has a valid email".

```typescript
// Entidade pura — zero dependência externa
class Order {
  private items: OrderItem[] = [];
  private status: OrderStatus = 'pending';

  addItem(product: Product, quantity: number): void {
    if (quantity <= 0) {
      throw new Error('Quantidade deve ser positiva');
    }
    this.items.push(new OrderItem(product, quantity));
  }

  getTotal(): number {
    return this.items.reduce(
      (sum, item) => sum + item.product.price * item.quantity,
      0
    );
  }

  confirm(): void {
    if (this.items.length === 0) {
      throw new Error('Pedido não pode ser confirmado sem itens');
    }
    this.status = 'confirmed';
  }
}
```