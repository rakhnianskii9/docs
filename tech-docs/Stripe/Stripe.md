# Stripe API для Node.js TypeScript

## Краткий обзор

Stripe Node.js библиотека предоставляет удобный доступ к Stripe API для серверных приложений, написанных на JavaScript/TypeScript. Эта документация содержит всю необходимую информацию для интеграции Stripe в ваше приложение.

## Установка

```bash
npm install stripe
```

или

```bash
yarn add stripe
```

## Требования

- Node.js 12 или выше
- TypeScript (для TypeScript проектов)

## Основная настройка

### JavaScript/CommonJS

```javascript
const stripe = require('stripe')('sk_test_...');

stripe.customers.create({
  email: 'customer@example.com',
})
  .then(customer => console.log(customer.id))
  .catch(error => console.error(error));
```

### TypeScript/ES Modules

```typescript
import Stripe from 'stripe';

const stripe = new Stripe('sk_test_...', {
  apiVersion: '2023-10-16', // Используйте последнюю версию API
});

const customer = await stripe.customers.create({
  email: 'customer@example.com',
});

console.log(customer.id);
```

## Конфигурация

### Инициализация с настройками

```typescript
import Stripe from 'stripe';
import { ProxyAgent } from 'https-proxy-agent';

const stripe = new Stripe('sk_test_...', {
  apiVersion: '2023-10-16',
  maxNetworkRetries: 3,
  timeout: 20000,
  telemetry: true,
  appInfo: {
    name: 'MyApp',
    version: '1.0.0',
    url: 'https://myapp.com',
  },
  // Для работы через прокси
  httpAgent: new ProxyAgent(process.env.HTTP_PROXY),
});
```

### Основные параметры конфигурации

| Параметр | Значение по умолчанию | Описание |
|----------|----------------------|----------|
| `apiVersion` | `null` | Версия API Stripe |
| `maxNetworkRetries` | `1` | Количество повторных попыток запроса |
| `timeout` | `80000` | Таймаут запроса в миллисекундах |
| `host` | `'api.stripe.com'` | Хост для запросов |
| `port` | `443` | Порт для запросов |
| `protocol` | `'https'` | Протокол (`https` или `http`) |
| `telemetry` | `true` | Отправка телеметрии |

## Использование с TypeScript

### Типизация

```typescript
import Stripe from 'stripe';

const stripe = new Stripe('sk_test_...', {
  apiVersion: '2023-10-16',
});

// Использование типов
const createCustomer = async (): Promise<Stripe.Customer> => {
  const params: Stripe.CustomerCreateParams = {
    email: 'customer@example.com',
    name: 'John Doe',
    description: 'Test customer',
  };

  const customer: Stripe.Customer = await stripe.customers.create(params);
  return customer;
};
```

### Работа с expandable полями

```typescript
const paymentIntent = await stripe.paymentIntents.retrieve(
  'pi_1234567890',
  {
    expand: ['customer', 'payment_method'],
  }
);

// Приведение типов для expanded полей
const customer = paymentIntent.customer as Stripe.Customer;
const paymentMethod = paymentIntent.payment_method as Stripe.PaymentMethod;

console.log(customer.email);
console.log(paymentMethod.type);
```

## Основные операции

### Работа с клиентами

```typescript
// Создание клиента
const customer = await stripe.customers.create({
  email: 'customer@example.com',
  name: 'John Doe',
  phone: '+1234567890',
  metadata: {
    order_id: '123456',
  },
});

// Получение клиента
const retrievedCustomer = await stripe.customers.retrieve(customer.id);

// Обновление клиента
const updatedCustomer = await stripe.customers.update(customer.id, {
  description: 'Updated customer description',
});

// Список клиентов
const customers = await stripe.customers.list({
  limit: 10,
  created: {
    gte: Math.floor(Date.now() / 1000) - 86400, // За последние 24 часа
  },
});
```

### Работа с платежами (Payment Intents)

```typescript
// Создание Payment Intent
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000, // $20.00
  currency: 'usd',
  customer: customer.id,
  payment_method_types: ['card'],
  metadata: {
    order_id: '123456',
  },
});

// Подтверждение платежа
const confirmedPayment = await stripe.paymentIntents.confirm(
  paymentIntent.id,
  {
    payment_method: 'pm_card_visa',
  }
);

// Получение платежа
const retrievedPayment = await stripe.paymentIntents.retrieve(
  paymentIntent.id
);

// Отмена платежа
const canceledPayment = await stripe.paymentIntents.cancel(
  paymentIntent.id
);
```

### Работа с подписками

```typescript
// Создание продукта
const product = await stripe.products.create({
  name: 'Monthly Subscription',
  type: 'service',
});

// Создание цены
const price = await stripe.prices.create({
  unit_amount: 2000,
  currency: 'usd',
  recurring: {
    interval: 'month',
  },
  product: product.id,
});

// Создание подписки
const subscription = await stripe.subscriptions.create({
  customer: customer.id,
  items: [
    {
      price: price.id,
    },
  ],
  payment_behavior: 'default_incomplete',
  payment_settings: {
    save_default_payment_method: 'on_subscription',
  },
  expand: ['latest_invoice.payment_intent'],
});
```

### Работа с вебхуками

```typescript
import { Request, Response } from 'express';

const endpointSecret = 'whsec_...'; // Ваш webhook secret

export const handleWebhook = (req: Request, res: Response) => {
  const sig = req.headers['stripe-signature'];

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(req.body, sig, endpointSecret);
  } catch (err) {
    console.log(`Webhook signature verification failed.`, err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Обработка события
  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      console.log('PaymentIntent succeeded:', paymentIntent.id);
      break;
    case 'customer.subscription.created':
      const subscription = event.data.object as Stripe.Subscription;
      console.log('Subscription created:', subscription.id);
      break;
    default:
      console.log(`Unhandled event type ${event.type}`);
  }

  res.json({ received: true });
};
```

## Обработка ошибок

```typescript
import Stripe from 'stripe';

try {
  const customer = await stripe.customers.create({
    email: 'invalid-email',
  });
} catch (error) {
  if (error instanceof Stripe.errors.StripeError) {
    console.error('Stripe error:', error.message);
    console.error('Error type:', error.type);
    console.error('Error code:', error.code);
    console.error('HTTP status:', error.statusCode);
  } else {
    console.error('Unknown error:', error);
  }
}
```

### Типы ошибок Stripe

```typescript
// Основные типы ошибок
try {
  // ... операция со Stripe
} catch (error) {
  if (error instanceof Stripe.errors.StripeCardError) {
    // Ошибка карты
    console.log('Card error:', error.message);
  } else if (error instanceof Stripe.errors.StripeRateLimitError) {
    // Превышение лимита запросов
    console.log('Rate limit error');
  } else if (error instanceof Stripe.errors.StripeInvalidRequestError) {
    // Некорректный запрос
    console.log('Invalid request:', error.message);
  } else if (error instanceof Stripe.errors.StripeAuthenticationError) {
    // Ошибка аутентификации
    console.log('Authentication error');
  } else if (error instanceof Stripe.errors.StripeConnectionError) {
    // Ошибка соединения
    console.log('Connection error');
  } else if (error instanceof Stripe.errors.StripeAPIError) {
    // Ошибка API
    console.log('API error');
  }
}
```

## Расширенные возможности

### Пагинация

```typescript
// Автоматическая пагинация с async/await
for await (const customer of stripe.customers.list()) {
  console.log(customer.email);
  // Обрабатывать каждого клиента
}

// Ручная пагинация
const listCustomers = async () => {
  const customers = await stripe.customers.list({
    limit: 100,
  });

  // Получить следующую страницу
  if (customers.has_more) {
    const nextCustomers = await stripe.customers.list({
      limit: 100,
      starting_after: customers.data[customers.data.length - 1].id,
    });
  }
};

// Получить все элементы (осторожно с большими списками)
const allCustomers = await stripe.customers.list({
  limit: 100,
}).autoPagingToArray({ limit: 10000 });
```

### Работа с метаданными

```typescript
// Добавление метаданных
const customer = await stripe.customers.create({
  email: 'customer@example.com',
  metadata: {
    order_id: '123456',
    user_id: 'user_789',
    source: 'website',
  },
});

// Поиск по метаданным
const customers = await stripe.customers.list({
  limit: 10,
  // Поиск не поддерживается напрямую через метаданные
  // Используйте Search API для этого
});

// Использование Search API
const searchResults = await stripe.customers.search({
  query: 'metadata["order_id"]:"123456"',
});
```

### Идемпотентность

```typescript
import { v4 as uuidv4 } from 'uuid';

const idempotencyKey = uuidv4();

const customer = await stripe.customers.create(
  {
    email: 'customer@example.com',
  },
  {
    idempotencyKey: idempotencyKey,
  }
);
```

### Работа с файлами

```typescript
import fs from 'fs';

// Загрузка файла
const file = await stripe.files.create({
  file: {
    data: fs.readFileSync('path/to/file.pdf'),
    name: 'file.pdf',
    type: 'application/pdf',
  },
  purpose: 'dispute_evidence',
});

// Получение файла
const retrievedFile = await stripe.files.retrieve(file.id);
```

## Stripe Connect

```typescript
// Создание подключенного аккаунта
const account = await stripe.accounts.create({
  type: 'express',
  country: 'US',
  email: 'business@example.com',
});

// Выполнение операций от имени подключенного аккаунта
const customer = await stripe.customers.create(
  {
    email: 'customer@example.com',
  },
  {
    stripeAccount: account.id,
  }
);

// Создание трансфера
const transfer = await stripe.transfers.create({
  amount: 1000,
  currency: 'usd',
  destination: account.id,
});
```

## Тестирование

### Тестовые карты

```typescript
// Успешная карта
const testCard = 'pm_card_visa';

// Карта с ошибкой
const declinedCard = 'pm_card_chargeDeclined';

// Тестовый платеж
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000,
  currency: 'usd',
  payment_method: testCard,
  confirm: true,
});
```

### Моковый webhook

```typescript
// Генерация тестового webhook события
const payload = {
  id: 'evt_test_webhook',
  object: 'event',
  type: 'payment_intent.succeeded',
  data: {
    object: {
      id: 'pi_test_123',
      object: 'payment_intent',
      amount: 2000,
      currency: 'usd',
      status: 'succeeded',
    },
  },
};

const payloadString = JSON.stringify(payload, null, 2);
const secret = 'whsec_test_secret';

const header = stripe.webhooks.generateTestHeaderString({
  payload: payloadString,
  secret,
});

const event = stripe.webhooks.constructEvent(payloadString, header, secret);
```

## Лучшие практики

### 1. Безопасность

```typescript
// Используйте переменные окружения для ключей
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

// Валидация webhook подписи
const verifyWebhook = (payload: string, signature: string) => {
  try {
    return stripe.webhooks.constructEvent(
      payload,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    throw new Error('Invalid webhook signature');
  }
};
```

### 2. Обработка ошибок

```typescript
const createCustomerSafely = async (email: string) => {
  try {
    return await stripe.customers.create({ email });
  } catch (error) {
    if (error instanceof Stripe.errors.StripeError) {
      // Логирование ошибки
      console.error('Stripe error:', {
        type: error.type,
        code: error.code,
        message: error.message,
        statusCode: error.statusCode,
      });
      
      // Возврат понятной ошибки пользователю
      throw new Error('Payment service error. Please try again.');
    }
    throw error;
  }
};
```

### 3. Конфигурация для продакшена

```typescript
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
  maxNetworkRetries: 3,
  timeout: 20000,
  telemetry: true,
  appInfo: {
    name: 'YourApp',
    version: process.env.APP_VERSION || '1.0.0',
    url: process.env.APP_URL,
  },
});
```

## Полезные ссылки

- [Официальная документация Stripe](https://stripe.com/docs)
- [Stripe API Reference](https://stripe.com/docs/api)
- [Stripe Node.js GitHub](https://github.com/stripe/stripe-node)
- [Stripe Dashboard](https://dashboard.stripe.com/)
- [Stripe Webhook Testing](https://stripe.com/docs/webhooks/test)

## Примеры кода

### Полный пример создания платежа

```typescript
import Stripe from 'stripe';
import express from 'express';

const app = express();
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

app.use(express.json());

// Создание Payment Intent
app.post('/create-payment-intent', async (req, res) => {
  try {
    const { amount, currency = 'usd', customer_id } = req.body;

    const paymentIntent = await stripe.paymentIntents.create({
      amount: amount * 100, // Конвертация в центы
      currency,
      customer: customer_id,
      automatic_payment_methods: {
        enabled: true,
      },
    });

    res.json({
      client_secret: paymentIntent.client_secret,
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Обработка webhook
app.post('/webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.log(`Webhook signature verification failed.`, err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      console.log('PaymentIntent succeeded:', paymentIntent.id);
      // Обновить заказ в базе данных
      break;
    default:
      console.log(`Unhandled event type ${event.type}`);
  }

  res.json({ received: true });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```