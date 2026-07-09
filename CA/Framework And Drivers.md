-  In this layer, we have all external details, such as web framework, ORM, Database Client, etc;
-  Also here, that we instantiate all previous interfaces in concrete classes;
```ts
import express from 'express';
import { Pool } from 'pg';
import nodemailer from 'nodemailer';

// Frameworks & Drivers
const app = express();
const db = new Pool({ connectionString: process.env.DATABASE_URL });
const mailer = nodemailer.createTransport({ /* ... */ });

// Interface Adapters
const orderRepository = new PostgresOrderRepository(db);
const emailService = new NodemailerEmailService(mailer);
const productRepository = new PostgresProductRepository(db);

// Use Case
const placeOrderUseCase = new PlaceOrderUseCase(
  orderRepository,
  emailService,
  productRepository
);

// Controller
const orderController = new OrderController(placeOrderUseCase);

// Framework conecta a rota ao controller
app.post('/orders', (req, res) => orderController.handlePost(req, res));

app.listen(3000);
```

