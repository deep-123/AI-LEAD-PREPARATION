# Design Patterns in Software Development
## Complete Guide for 10 YOE Full-Stack Developer → Tech Lead → AI Lead / Solution Architect

**Primary language:** JavaScript / Node.js  
**Secondary language:** Python  
**Focus:** Production development + System Design + Technical Leadership + Interviews

---

# 1. What is a Design Pattern?

### What is it?

A **design pattern is a proven, reusable solution approach to a commonly occurring software design problem.**

It is **not a library, framework, or copy-paste code**.

Think of it as:

> **Problem → Proven design → Reusable solution structure**

For example:

You have multiple payment providers:

```text
Order Service
     |
     +---- Stripe
     +---- Razorpay
     +---- PayPal
```

Instead of writing:

```javascript
if (provider === "stripe") {
   // Stripe logic
} else if (provider === "razorpay") {
   // Razorpay logic
}
```

everywhere, you can use a **Factory + Strategy + Adapter** combination.

---

# 2. Why Do We Need Design Patterns?

### What?

Patterns help us structure software so that it is easier to:

- Maintain
- Extend
- Test
- Understand
- Reuse
- Scale
- Replace components

### Why?

Without patterns, large applications often become:

```text
Business Logic
      |
      +--- Database
      +--- API
      +--- Payment
      +--- Email
      +--- Logging
      +--- Third-party SDK
      +--- Authentication
```

Everything becomes tightly coupled.

With good design:

```text
Business Logic
      |
      +--- Interface / Abstraction
              |
       +------+------+
       |             |
    Stripe        Razorpay
```

You can change implementation without changing business logic.

### How?

Patterns usually help us achieve:

- **Loose coupling**
- **High cohesion**
- **Separation of concerns**
- **Open/Closed Principle**
- **Dependency Inversion**
- **Testability**
- **Extensibility**

---

# 3. Categories of Design Patterns

The original **Gang of Four (GoF)** patterns are divided into three categories.

| Category | Purpose | Examples |
|---|---|---|
| 🏗️ Creational | Object creation | Factory, Builder, Singleton |
| 🧱 Structural | Object/class composition | Adapter, Decorator, Facade |
| 🔄 Behavioral | Communication/behavior | Strategy, Observer, Command |

For modern enterprise applications, we also commonly use:

| Type | Examples |
|---|---|
| Enterprise/Application | Repository, Dependency Injection |
| Concurrency/Architectural | Reactor |

> **Interview Tip:** Repository, Dependency Injection, and Reactor are important modern patterns/techniques, but they are **not part of the original 23 GoF design patterns**.

---

# 4. Pattern #1 — Factory

## Category: 🏗️ Creational

### What is Factory?

Factory centralizes object creation.

Instead of the client knowing:

```javascript
new StripePayment()
new RazorpayPayment()
new PaypalPayment()
```

the client asks:

```javascript
PaymentFactory.create("stripe")
```

### Why use it?

- Hide object creation logic
- Avoid repeated `if/else`
- Centralize configuration
- Make adding implementations easier
- Reduce coupling

### How does it work?

```text
             Client
                |
                v
          PaymentFactory
          /      |      \
         v       v       v
      Stripe  Razorpay  PayPal
```

### Node.js Example

```javascript
class StripePayment {
    pay(amount) {
        console.log(`Stripe payment: ₹${amount}`);
    }
}

class RazorpayPayment {
    pay(amount) {
        console.log(`Razorpay payment: ₹${amount}`);
    }
}

class PaymentFactory {
    static create(provider) {
        switch (provider) {
            case "stripe":
                return new StripePayment();

            case "razorpay":
                return new RazorpayPayment();

            default:
                throw new Error("Unsupported payment provider");
        }
    }
}

const payment = PaymentFactory.create("stripe");

payment.pay(1000);
```

### Real-world use cases

- Payment provider selection
- Database driver creation
- Cloud provider clients
- Notification providers
- AI model providers
- Logger creation
- File storage providers

For an AI platform:

```text
AIProviderFactory
       |
       +--- OpenAI
       +--- Gemini
       +--- Anthropic
       +--- Azure OpenAI
```

### When to use

- Object creation is complex
- Multiple implementations exist
- Selection depends on runtime configuration
- You want to hide concrete classes

### When NOT to use

Don't create a Factory for:

```javascript
new User()
```

if there is no meaningful creation complexity.

### Interview Answer 🎤

> "Factory is a creational pattern that encapsulates object creation. Instead of tightly coupling the client to concrete implementations, the client asks a factory for the required implementation. I've used this approach for provider selection, such as payment, notification, cloud, or AI providers."

### Interview follow-up

**Factory vs DI?**

> Factory decides **which object to create**. DI provides the dependency to the consumer. They can also work together.

---

# 5. Pattern #2 — Strategy

## Category: 🔄 Behavioral

### What is Strategy?

Strategy allows us to define multiple algorithms/behaviors and select one at runtime.

### Why?

Instead of:

```javascript
if (customer === "premium") {
   ...
} else if (customer === "gold") {
   ...
} else if (customer === "silver") {
   ...
}
```

we create independent strategies.

### How?

```text
                 OrderService
                      |
                DiscountStrategy
                 /      |      \
                /       |       \
          Premium      Gold     Silver
```

### Node.js

```javascript
class NoDiscount {
    calculate(amount) {
        return amount;
    }
}

class PremiumDiscount {
    calculate(amount) {
        return amount * 0.8;
    }
}

class GoldDiscount {
    calculate(amount) {
        return amount * 0.9;
    }
}

class OrderService {
    constructor(discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    calculatePrice(amount) {
        return this.discountStrategy.calculate(amount);
    }
}

const service =
    new OrderService(new PremiumDiscount());

console.log(service.calculatePrice(1000));
```

Output:

```text
800
```

### Real-world use cases

- Pricing rules
- Discount calculation
- Authentication mechanisms
- Payment routing
- Tax calculation
- Shipping calculation
- Search algorithms
- AI model selection
- Retry strategies

### AI Lead example

```text
Request
   |
AI Routing Strategy
   |
   +--- Cheap Model
   +--- Fast Model
   +--- High Quality Model
   +--- Private Model
```

The routing strategy can change based on:

- Cost
- Latency
- Token limit
- Data sensitivity
- Quality requirement

### When to use

Use Strategy when:

- You have interchangeable algorithms
- Business rules keep changing
- `if/else` keeps growing
- Runtime selection is required

### When NOT to use

If there are only two very simple conditions:

```javascript
const price = isPremium ? 800 : 1000;
```

don't create five classes.

### Interview Answer 🎤

> "Strategy encapsulates interchangeable algorithms behind a common interface. I use it when business behavior varies independently from the main workflow, such as pricing, payment routing, retry policies, or AI model selection."

### Key benefit

**Open for extension, closed for modification.**

---

# 6. Pattern #3 — Adapter

## Category: 🧱 Structural

### What is Adapter?

Adapter converts one interface into another interface expected by your application.

### Real problem

Your application expects:

```javascript
payment.pay(amount)
```

But Stripe provides:

```javascript
stripe.charge(amount)
```

Razorpay provides:

```javascript
razorpay.makePayment(amount)
```

### Solution

```text
Application
     |
     v
Payment Interface
     |
 +---+---------+
 |             |
StripeAdapter  RazorpayAdapter
 |             |
Stripe SDK     Razorpay SDK
```

### Node.js

```javascript
class StripeSDK {
    charge(amount) {
        console.log(`Stripe charged ₹${amount}`);
    }
}

class StripeAdapter {
    constructor() {
        this.stripe = new StripeSDK();
    }

    pay(amount) {
        this.stripe.charge(amount);
    }
}

class RazorpaySDK {
    makePayment(amount) {
        console.log(`Razorpay charged ₹${amount}`);
    }
}

class RazorpayAdapter {
    constructor() {
        this.razorpay = new RazorpaySDK();
    }

    pay(amount) {
        this.razorpay.makePayment(amount);
    }
}

function checkout(paymentProvider) {
    paymentProvider.pay(1000);
}

checkout(new StripeAdapter());
checkout(new RazorpayAdapter());
```

### Real-world use cases

- Third-party APIs
- Legacy systems
- Cloud SDKs
- Payment gateways
- CRM integrations
- AI provider APIs
- Different database clients

### When to use

Use Adapter when:

> **Existing component works, but its interface doesn't match your application.**

### When NOT to use

Don't create adapters around every simple class.

### Interview Answer 🎤

> "Adapter allows incompatible interfaces to work together. In production systems, I commonly use adapters around third-party SDKs so that business logic depends on our internal interface rather than vendor-specific APIs."

### Senior-level point

Adapter also helps with **vendor lock-in**.

Today:

```text
Our application → StripeAdapter → Stripe
```

Tomorrow:

```text
Our application → RazorpayAdapter → Razorpay
```

Business logic remains unchanged.

---

# 7. Pattern #4 — Observer / Pub-Sub

## Category: 🔄 Behavioral

### What is Observer?

Observer allows one object to notify multiple interested consumers when something happens.

### Example

An order is created.

Multiple things need to happen:

```text
Order Created
     |
     +---- Email
     +---- Inventory
     +---- Audit
     +---- Analytics
     +---- Notification
```

The Order Service shouldn't directly depend on all of them.

### How?

```text
              EventBus
             /   |   \
            /    |    \
        Email Inventory Audit
```

### Node.js

```javascript
class EventBus {
    constructor() {
        this.listeners = {};
    }

    subscribe(event, callback) {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }

        this.listeners[event].push(callback);
    }

    publish(event, data) {
        const callbacks = this.listeners[event] || [];

        callbacks.forEach(callback => {
            callback(data);
        });
    }
}

const eventBus = new EventBus();

eventBus.subscribe("ORDER_CREATED", order => {
    console.log("Sending email", order.id);
});

eventBus.subscribe("ORDER_CREATED", order => {
    console.log("Updating inventory", order.id);
});

eventBus.subscribe("ORDER_CREATED", order => {
    console.log("Writing audit log", order.id);
});

eventBus.publish("ORDER_CREATED", {
    id: "ORD-101"
});
```

### Real-world use cases

- Kafka events
- RabbitMQ
- SNS/SQS
- Domain events
- WebSocket notifications
- Audit systems
- Analytics
- Microservices communication

### When to use

Use when:

- Many consumers react to one event
- Producer shouldn't know consumers
- Asynchronous processing is useful

### When NOT to use

Avoid events when you need an immediate synchronous response from another component.

### Interview Answer 🎤

> "Observer creates a one-to-many relationship where subscribers react to events published by a producer. In distributed systems, this concept maps naturally to event-driven architecture using Kafka, SNS/SQS, RabbitMQ, etc."

### Important interview distinction

**Observer ≠ Kafka**

Observer is the design concept.

Kafka is an infrastructure technology that can implement event-driven communication at scale.

---

# 8. Pattern #5 — Decorator

## Category: 🧱 Structural

### What is Decorator?

Decorator dynamically adds behavior to an existing object without modifying its original implementation.

### Example

Base service:

```text
Payment
```

Add:

```text
Logging
Metrics
Authorization
Caching
Retry
```

### Structure

```text
LoggingDecorator
       |
MetricsDecorator
       |
PaymentService
```

### Node.js

```javascript
class PaymentService {
    pay(amount) {
        console.log(`Payment processed: ₹${amount}`);
    }
}

class LoggingDecorator {
    constructor(service) {
        this.service = service;
    }

    pay(amount) {
        console.log("Payment started");

        this.service.pay(amount);

        console.log("Payment completed");
    }
}

const payment = new PaymentService();

const loggedPayment =
    new LoggingDecorator(payment);

loggedPayment.pay(1000);
```

### Real-world use cases

- Logging
- Metrics
- Caching
- Authorization
- Retry
- Tracing
- Rate limiting

### Interview Answer 🎤

> "Decorator lets us add cross-cutting behavior around an existing implementation without changing the original class. It is useful for logging, metrics, caching, authorization, retries, and observability."

### Decorator vs Adapter

| Adapter | Decorator |
|---|---|
| Changes interface | Usually preserves interface |
| Makes incompatible things compatible | Adds behavior |
| Integration-focused | Behavior-focused |

---

# 9. Pattern #6 — Facade

## Category: 🧱 Structural

### What is Facade?

Facade provides a simple interface over a complex subsystem.

### Without Facade

```text
Client
 |
 +--- Validate
 +--- Payment
 +--- Inventory
 +--- Email
 +--- Audit
```

### With Facade

```text
Client
  |
  v
OrderFacade
  |
  +--- Validation
  +--- Payment
  +--- Inventory
  +--- Email
  +--- Audit
```

### Node.js

```javascript
class PaymentService {
    pay(amount) {
        console.log("Payment successful");
    }
}

class InventoryService {
    reserve(product) {
        console.log(`Reserved ${product}`);
    }
}

class EmailService {
    send(email) {
        console.log(`Email sent to ${email}`);
    }
}

class OrderFacade {
    constructor(payment, inventory, email) {
        this.payment = payment;
        this.inventory = inventory;
        this.email = email;
    }

    placeOrder(order) {
        this.payment.pay(order.amount);

        this.inventory.reserve(order.product);

        this.email.send(order.email);

        console.log("Order completed");
    }
}

const facade = new OrderFacade(
    new PaymentService(),
    new InventoryService(),
    new EmailService()
);

facade.placeOrder({
    amount: 1000,
    product: "Laptop",
    email: "user@example.com"
});
```

### Real-world use cases

- API Gateway
- Service layer
- Checkout APIs
- Authentication facade
- Cloud service wrapper
- Complex business workflows

### Interview Answer 🎤

> "Facade provides a simplified entry point to a complex subsystem. I use it at service or API boundaries where consumers shouldn't need to understand all internal orchestration."

### Senior-level insight

Facade is particularly useful at **microservice boundaries**.

---

# 10. Pattern #7 — Builder

## Category: 🏗️ Creational

### What is Builder?

Builder constructs complex objects step by step.

### Problem

Imagine:

```javascript
new User(
    name,
    email,
    phone,
    address,
    role,
    permissions,
    preferences,
    metadata
);
```

This becomes difficult to understand.

### Builder

```javascript
class UserBuilder {
    constructor() {
        this.user = {};
    }

    setName(name) {
        this.user.name = name;
        return this;
    }

    setEmail(email) {
        this.user.email = email;
        return this;
    }

    setRole(role) {
        this.user.role = role;
        return this;
    }

    build() {
        return this.user;
    }
}

const user = new UserBuilder()
    .setName("Deepak")
    .setEmail("deepak@example.com")
    .setRole("Tech Lead")
    .build();

console.log(user);
```

### Real-world use cases

- Complex configuration
- HTTP request builders
- Query builders
- Test data
- Cloud infrastructure configuration
- AI request configuration

Example:

```javascript
const request = new AIRequestBuilder()
    .setModel("gpt")
    .setTemperature(0.2)
    .setMaxTokens(1000)
    .enableStreaming()
    .build();
```

### Interview Answer 🎤

> "Builder separates complex object construction from its final representation. I use it when objects have many optional parameters or configuration steps."

---

# 11. Pattern #8 — Singleton

## Category: 🏗️ Creational

### What is Singleton?

Singleton ensures that a particular class has only one shared instance within the intended scope.

### Common use cases

- Configuration
- Logger
- Metrics
- Connection managers
- Application-wide services

### Node.js

```javascript
class Logger {
    constructor() {
        if (Logger.instance) {
            return Logger.instance;
        }

        Logger.instance = this;
    }

    log(message) {
        console.log(`[LOG] ${message}`);
    }
}

const logger1 = new Logger();
const logger2 = new Logger();

console.log(logger1 === logger2);
```

Output:

```text
true
```

### Important interview warning

Singleton is often overused.

Bad:

```text
Everything → Singleton
```

This creates:

- Hidden dependencies
- Global state
- Difficult testing
- Tight coupling

### Interview Answer 🎤

> "Singleton guarantees one shared instance within a defined scope. It can be useful for stateless infrastructure such as configuration or logging, but I avoid using it as a general-purpose global state mechanism because it hurts testability and dependency management."

### Senior-level answer

In Node.js, module caching can already provide singleton-like behavior.

---

# 12. Pattern #9 — Repository

## Category: 🏢 Enterprise / Application Pattern

> **Repository is not one of the original GoF 23 patterns.**

### What is Repository?

Repository abstracts data-access logic from business logic.

### Without Repository

```text
OrderService
   |
   +--- SQL
   +--- MongoDB
   +--- Redis
```

### With Repository

```text
OrderService
     |
OrderRepository
     |
 +---+---------+
 |             |
MongoDB       PostgreSQL
```

### Node.js

```javascript
class OrderRepository {
    async save(order) {
        console.log("Saving order:", order);
    }

    async findById(id) {
        console.log("Finding order:", id);

        return {
            id,
            amount: 1000
        };
    }
}

class OrderService {
    constructor(orderRepository) {
        this.orderRepository = orderRepository;
    }

    async createOrder(order) {
        return this.orderRepository.save(order);
    }
}
```

### Why?

- Separates business logic from persistence
- Easier unit testing
- Database migration becomes easier
- Centralizes data access
- Prevents SQL/ORM code spreading across services

### Interview Answer 🎤

> "Repository abstracts persistence concerns behind a domain-oriented interface. It keeps business services independent from database implementation and makes testing and persistence changes easier."

### Important distinction

Repository is **not simply a DAO**.

A Repository is generally expressed around domain/business concepts, whereas DAO tends to be closer to raw persistence operations.

---

# 13. Pattern #10 — Command

## Category: 🔄 Behavioral

### What is Command?

Command encapsulates an action/request as an object.

Instead of:

```javascript
orderService.cancelOrder(id);
```

we can represent:

```text
CancelOrderCommand
```

### Structure

```text
Command
   |
   +--- Execute
   |
Receiver
   |
   +--- Business Logic
```

### Node.js

```javascript
class CreateOrderCommand {
    constructor(orderService, order) {
        this.orderService = orderService;
        this.order = order;
    }

    async execute() {
        return this.orderService.createOrder(this.order);
    }
}

class OrderService {
    async createOrder(order) {
        console.log("Creating order", order.id);

        return order;
    }
}

const service = new OrderService();

const command = new CreateOrderCommand(
    service,
    { id: "ORD-101" }
);

command.execute();
```

### Real-world use cases

- Job queues
- Undo/redo
- Background processing
- Workflow engines
- Transaction commands
- CQRS
- Task execution
- Event-driven systems

### Interview Answer 🎤

> "Command encapsulates an operation as an object. This is useful when operations need to be queued, logged, retried, scheduled, audited, or undone."

---

# 14. Pattern #11 — Dependency Injection

## Category: 🏢 Enterprise / Application Pattern

> DI is not one of the original GoF 23 patterns.

### What is DI?

Dependency Injection means:

> **A class receives its dependencies from outside instead of creating them internally.**

### Bad

```javascript
class OrderService {
    constructor() {
        this.database = new MongoDatabase();
        this.email = new EmailService();
    }
}
```

The class controls everything.

### Better

```javascript
class OrderService {
    constructor(database, emailService) {
        this.database = database;
        this.emailService = emailService;
    }
}
```

Now:

```javascript
const service = new OrderService(
    mongoDatabase,
    emailService
);
```

### Why?

DI provides:

- Loose coupling
- Testability
- Replaceability
- Better architecture
- Clear dependencies

### Testing becomes easy

```javascript
const mockDatabase = {
    save: async () => true
};

const mockEmail = {
    send: async () => true
};

const service =
    new OrderService(
        mockDatabase,
        mockEmail
    );
```

### Interview Answer 🎤

> "Dependency Injection means supplying a component's dependencies from outside rather than constructing them internally. It reduces coupling and improves testability and flexibility. I typically use constructor injection because dependencies become explicit."

### DI + Factory

These often work together:

```text
Composition Root
       |
       +--- Factory creates implementation
       |
       +--- DI injects it
       |
       v
   Business Service
```

---

# 15. Pattern #12 — Reactor

## Category: ⚡ Concurrency / Architectural Pattern

> Reactor is not one of the GoF 23 patterns.

### What is Reactor?

Reactor is an event-driven concurrency pattern where:

1. Events arrive
2. An event loop monitors them
3. Appropriate handlers are invoked
4. Work proceeds without blocking the main event loop

### Node.js architecture

```text
                Node.js
                   |
              Event Loop
                   |
       +-----------+-----------+
       |           |           |
     HTTP        Timer       I/O
       |           |           |
       +-----------+-----------+
                   |
              Event Handler
```

### Example

```javascript
const fs = require("fs");

console.log("1. Start");

fs.readFile("file.txt", "utf8", (err, data) => {
    console.log("3. File read completed");
});

console.log("2. Continue without waiting");
```

Typical output:

```text
1. Start
2. Continue without waiting
3. File read completed
```

The main JavaScript execution doesn't block while the file operation is being handled.

### Why is Reactor important for Node.js?

Node.js is heavily based on:

- Event loop
- Non-blocking I/O
- Event-driven programming
- Callbacks
- Promises
- Async/await

### Python equivalent

Python can implement similar event-driven behavior with `asyncio`.

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)
    print("Data received")

async def main():
    print("Start")

    task = asyncio.create_task(fetch_data())

    print("Continue")

    await task

asyncio.run(main())
```

### Real-world use cases

- HTTP servers
- WebSocket servers
- Network applications
- High-concurrency APIs
- Message consumers
- Event-driven systems

### Interview Answer 🎤

> "Reactor is an event-driven concurrency pattern where an event loop dispatches incoming events to handlers. Node.js is strongly based on this model, using its event loop and non-blocking I/O to handle many concurrent operations efficiently."

### Important interview point

**Async does not automatically mean parallel.**

Node.js JavaScript execution is primarily single-threaded, while I/O and certain CPU-heavy operations can be handled outside the main JavaScript execution path through the underlying runtime/platform mechanisms.

---

# 16. Complete Interconnected Node.js Example

The following example combines multiple patterns into one realistic application.

It demonstrates:

- Factory
- Strategy
- Adapter
- Repository
- Observer/EventBus
- Decorator
- Facade
- Dependency Injection

## Architecture

```text
                    API / Controller
                           |
                           v
                    OrderFacade
                           |
                           v
                     OrderService
                           |
       +-------------------+-------------------+
       |                   |                   |
       v                   v                   v
   Repository        PaymentFactory       Strategy
       |                   |                   |
       v                   v                   v
    Database          Adapter             Discount
                           |
                    +------+------+
                    |             |
                 Stripe       Razorpay

OrderService
     |
     v
 EventBus
  /  |  \
 /   |   \
Email Inventory Audit
```

---

## Complete runnable Node.js code

> The code block below is intentionally kept as a single runnable file so you can paste it directly into VS Code and execute it with Node.js.

```javascript
// design-patterns-demo.js

// =====================================================
// 1. ADAPTER PATTERN
// =====================================================

class StripeSDK {
    charge(amount) {
        console.log(`Stripe charged ₹${amount}`);
    }
}

class RazorpaySDK {
    makePayment(amount) {
        console.log(`Razorpay charged ₹${amount}`);
    }
}

class StripeAdapter {
    constructor() {
        this.stripe = new StripeSDK();
    }

    pay(amount) {
        this.stripe.charge(amount);
    }
}

class RazorpayAdapter {
    constructor() {
        this.razorpay = new RazorpaySDK();
    }

    pay(amount) {
        this.razorpay.makePayment(amount);
    }
}


// =====================================================
// 2. FACTORY PATTERN
// =====================================================

class PaymentFactory {

    static create(provider) {

        switch (provider) {

            case "stripe":
                return new StripeAdapter();

            case "razorpay":
                return new RazorpayAdapter();

            default:
                throw new Error(
                    `Unsupported provider: ${provider}`
                );
        }
    }
}


// =====================================================
// 3. STRATEGY PATTERN
// =====================================================

class NoDiscountStrategy {

    calculate(amount) {
        return amount;
    }
}

class PremiumDiscountStrategy {

    calculate(amount) {
        return amount * 0.80;
    }
}

class GoldDiscountStrategy {

    calculate(amount) {
        return amount * 0.90;
    }
}


// =====================================================
// 4. REPOSITORY PATTERN
// =====================================================

class OrderRepository {

    constructor() {
        this.orders = new Map();
    }

    async save(order) {

        this.orders.set(order.id, order);

        console.log(
            `Repository: Order ${order.id} saved`
        );

        return order;
    }

    async findById(id) {

        return this.orders.get(id);
    }
}


// =====================================================
// 5. OBSERVER / EVENT BUS
// =====================================================

class EventBus {

    constructor() {
        this.listeners = {};
    }

    subscribe(event, callback) {

        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }

        this.listeners[event].push(callback);
    }

    publish(event, data) {

        const callbacks =
            this.listeners[event] || [];

        callbacks.forEach(callback => {
            callback(data);
        });
    }
}


// =====================================================
// EVENT CONSUMERS
// =====================================================

class EmailService {

    sendOrderConfirmation(order) {

        console.log(
            `Email: Confirmation sent for ${order.id}`
        );
    }
}

class InventoryService {

    reserve(order) {

        console.log(
            `Inventory: Reserved ${order.product}`
        );
    }
}

class AuditService {

    log(order) {

        console.log(
            `Audit: Order ${order.id} created`
        );
    }
}


// =====================================================
// 6. ORDER SERVICE
// =====================================================

class OrderService {

    constructor(
        repository,
        paymentProvider,
        discountStrategy,
        eventBus
    ) {
        this.repository = repository;
        this.paymentProvider = paymentProvider;
        this.discountStrategy = discountStrategy;
        this.eventBus = eventBus;
    }

    async createOrder(order) {

        // Calculate final amount
        const finalAmount =
            this.discountStrategy
                .calculate(order.amount);

        console.log(
            `Original amount: ₹${order.amount}`
        );

        console.log(
            `Final amount: ₹${finalAmount}`
        );

        // Payment
        this.paymentProvider.pay(
            finalAmount
        );

        // Save
        const savedOrder =
            await this.repository.save({
                ...order,
                finalAmount
            });

        // Publish event
        this.eventBus.publish(
            "ORDER_CREATED",
            savedOrder
        );

        return savedOrder;
    }
}


// =====================================================
// 7. DECORATOR PATTERN
// =====================================================

class LoggingOrderService {

    constructor(orderService) {
        this.orderService = orderService;
    }

    async createOrder(order) {

        console.log(
            `[LOG] Creating order ${order.id}`
        );

        const result =
            await this.orderService.createOrder(
                order
            );

        console.log(
            `[LOG] Order ${order.id} completed`
        );

        return result;
    }
}


// =====================================================
// 8. FACADE PATTERN
// =====================================================

class OrderFacade {

    constructor(orderService) {
        this.orderService = orderService;
    }

    async placeOrder(order) {

        console.log(
            "\n===== ORDER START ====="
        );

        const result =
            await this.orderService
                .createOrder(order);

        console.log(
            "===== ORDER END =====\n"
        );

        return result;
    }
}


// =====================================================
// 9. DEPENDENCY INJECTION
// =====================================================

// Composition Root
// This is where we assemble the application.

const repository =
    new OrderRepository();

const paymentProvider =
    PaymentFactory.create("stripe");

const discountStrategy =
    new PremiumDiscountStrategy();

const eventBus =
    new EventBus();

const emailService =
    new EmailService();

const inventoryService =
    new InventoryService();

const auditService =
    new AuditService();


// Register observers

eventBus.subscribe(
    "ORDER_CREATED",
    order =>
        emailService
            .sendOrderConfirmation(order)
);

eventBus.subscribe(
    "ORDER_CREATED",
    order =>
        inventoryService
            .reserve(order)
);

eventBus.subscribe(
    "ORDER_CREATED",
    order =>
        auditService
            .log(order)
);


// Create core service

const orderService =
    new OrderService(
        repository,
        paymentProvider,
        discountStrategy,
        eventBus
    );


// Decorate service

const loggedOrderService =
    new LoggingOrderService(
        orderService
    );


// Create Facade

const orderFacade =
    new OrderFacade(
        loggedOrderService
    );


// =====================================================
// APPLICATION EXECUTION
// =====================================================

async function main() {

    const order = {

        id: "ORD-1001",

        product: "MacBook Pro",

        amount: 100000,

        email: "customer@example.com"
    };

    await orderFacade.placeOrder(order);
}

main();
```

---

# 17. What Patterns Are Working Together?

This is particularly important for **Tech Lead / Architect interviews**.

### Factory

```text
PaymentFactory
```

decides which payment implementation to create.

### Adapter

```text
StripeAdapter
RazorpayAdapter
```

normalizes third-party SDK interfaces.

### Strategy

```text
PremiumDiscountStrategy
GoldDiscountStrategy
NoDiscountStrategy
```

controls changing business rules.

### Repository

```text
OrderRepository
```

isolates persistence.

### Observer

```text
EventBus
```

decouples event producers from consumers.

### Decorator

```text
LoggingOrderService
```

adds logging without modifying `OrderService`.

### Facade

```text
OrderFacade
```

provides a simple API over the workflow.

### Dependency Injection

```text
OrderService(
    repository,
    paymentProvider,
    discountStrategy,
    eventBus
)
```

provides dependencies from outside.

---

# 18. Factory vs Strategy

This is a **very common interview question**.

| Factory | Strategy |
|---|---|
| Creational | Behavioral |
| Creates objects | Selects behavior |
| "Which object?" | "Which algorithm?" |
| Often used during setup | Often used during execution |
| Hides construction | Encapsulates behavior |

Example:

```text
Factory:
Which payment provider?

Strategy:
Which discount algorithm?
```

They can work together.

---

# 19. Adapter vs Facade

Another common interview question.

| Adapter | Facade |
|---|---|
| Makes incompatible interfaces compatible | Simplifies a complex subsystem |
| Usually wraps one external component | Often coordinates multiple components |
| Integration problem | Complexity problem |

Think:

```text
Adapter:
"Make them speak the same language."

Facade:
"Give me one simple door to the whole building."
```

---

# 20. Decorator vs Adapter

| Decorator | Adapter |
|---|---|
| Adds behavior | Changes interface |
| Usually preserves interface | Converts interface |
| Logging, metrics, caching | Third-party integration |

Example:

```text
Adapter:
Stripe → Our Payment Interface

Decorator:
Payment Service → Logging + Metrics
```

---

# 21. Observer vs Command

| Observer | Command |
|---|---|
| React to an event | Represent an action |
| Publisher → Subscribers | Command → Receiver |
| Event-driven | Action-driven |
| Kafka-style architecture | Job/task/workflow style |

Example:

```text
Observer:
ORDER_CREATED
      ↓
Email
Inventory
Audit
```

Command:

```text
CreateOrderCommand
      ↓
execute()
      ↓
OrderService
```

---

# 22. Factory vs Dependency Injection

### Factory

```text
I need an object.
Which implementation should I create?
```

### DI

```text
Here is the object you need.
```

Factory:

```javascript
const payment =
    PaymentFactory.create("stripe");
```

DI:

```javascript
const service =
    new OrderService(
        repository,
        payment,
        strategy
    );
```

### Senior answer

> "Factory controls object creation and selection, whereas Dependency Injection controls dependency provision. In production architecture, I often use Factory and DI together."

---

# 23. Design Patterns and SOLID

Design patterns become much more useful when combined with SOLID.

### Single Responsibility

```text
OrderService
PaymentService
EmailService
InventoryService
```

Each has a focused responsibility.

### Open/Closed

Adding:

```text
PayPal
```

should ideally not require changing all business logic.

### Liskov Substitution

Implementations should behave according to the expected abstraction.

```text
Payment
 ├── Stripe
 └── Razorpay
```

### Interface Segregation

Avoid huge interfaces.

Bad:

```text
Payment + Email + Inventory + Reporting
```

Better:

```text
Payment
Email
Inventory
Reporting
```

### Dependency Inversion

High-level business logic should depend on abstractions rather than concrete infrastructure.

```text
OrderService
      |
 Payment Interface
      |
 +----+----+
Stripe   Razorpay
```

---

# 24. Design Patterns in Microservices

Patterns become even more useful in distributed systems.

Example:

```text
                API Gateway
                     |
               Order Service
                     |
       +-------------+-------------+
       |             |             |
    Payment       Inventory      User
       |
    Adapter
       |
 Payment Provider
```

Events:

```text
Order Created
      |
      +---- Kafka
             |
       +-----+------+
       |            |
    Inventory     Email
```

Patterns involved:

- Facade → API/service boundary
- Adapter → external systems
- Strategy → business rules
- Factory → provider selection
- Observer → events
- Repository → persistence
- DI → dependency management
- Decorator → cross-cutting concerns

---

# 25. Design Patterns in AI Systems

For an **AI Lead / Solution Architect**, this is particularly valuable.

## AI Provider Factory

```text
AIProviderFactory
      |
      +--- OpenAI
      +--- Gemini
      +--- Anthropic
      +--- Azure OpenAI
```

## AI Provider Adapter

Normalize different SDKs:

```text
Our Interface

generate(prompt)

       |
       +--- OpenAI Adapter
       +--- Gemini Adapter
       +--- Anthropic Adapter
```

## Model Routing Strategy

```text
ModelRoutingStrategy
       |
       +--- CostOptimized
       +--- LatencyOptimized
       +--- QualityOptimized
       +--- PrivacyOptimized
```

## AI Decorator

```text
LLM Service
    |
    +--- Logging
    +--- Token Metrics
    +--- Retry
    +--- Caching
    +--- Tracing
```

## AI Facade

```text
AIApplicationFacade
       |
       +--- Prompt validation
       +--- Model routing
       +--- RAG
       +--- Guardrails
       +--- LLM
       +--- Output validation
```

This is a strong architecture-level interview discussion.

---

# 26. How to Answer Design Pattern Questions in Interviews

Use this framework:

### Step 1 — Definition

> "Strategy is a behavioral design pattern..."

### Step 2 — Problem

> "It solves the problem of having multiple interchangeable algorithms..."

### Step 3 — Solution

> "We encapsulate each algorithm behind a common interface..."

### Step 4 — Real-world example

> "For example, payment routing or AI model selection..."

### Step 5 — Production relevance

> "This reduces conditional logic and allows new strategies to be added without modifying the main workflow."

### Step 6 — Trade-off

> "The trade-off is that we introduce additional abstractions/classes, so I wouldn't use it for trivial logic."

### Step 7 — Alternative

> "For a simple condition, a conditional expression may be more appropriate."

This makes your answer sound like a **Senior Engineer / Tech Lead**, rather than someone who only memorized definitions.

---

# 27. What Interviewers Usually Want to Hear

Don't only say:

> "Factory creates objects."

Instead say:

> "I use Factory when object creation or implementation selection is variable. For example, in a payment system, the business service should not know whether Stripe or Razorpay is being instantiated. Factory centralizes that decision, while Adapter normalizes provider APIs. I combine that with DI so the business service remains loosely coupled and testable."

That demonstrates:

- Pattern knowledge
- Architecture knowledge
- SOLID
- Production experience
- Trade-off awareness
- Design thinking

---

# 28. When NOT to Use Design Patterns

This is one of the most important Tech Lead concepts.

Don't use patterns simply because you know them.

### Bad architecture

```text
Simple requirement
      |
      +--- Factory
      +--- Abstract Factory
      +--- Builder
      +--- Strategy
      +--- Decorator
      +--- Facade
      +--- Observer
      +--- Command
```

This is overengineering.

### Better principle

> **Use the simplest design that handles today's complexity while allowing reasonable future evolution.**

Patterns should solve a real problem.

---

# 29. Quick Cheat Sheet

| Pattern | Category | Main Purpose |
|---|---|---|
| Factory | Creational | Object creation |
| Builder | Creational | Complex object construction |
| Singleton | Creational | One shared instance |
| Adapter | Structural | Interface compatibility |
| Decorator | Structural | Add behavior |
| Facade | Structural | Simplify complexity |
| Strategy | Behavioral | Interchangeable algorithms |
| Observer | Behavioral | Event notification |
| Command | Behavioral | Encapsulate action |
| Repository | Enterprise | Abstract persistence |
| Dependency Injection | Enterprise/Application | Provide dependencies externally |
| Reactor | Concurrency/Architectural | Event-driven processing |

---

# 30. One-Line Memory Trick

Remember them like this:

```text
FACTORY
"Which object should I create?"

BUILDER
"How do I construct this complex object?"

SINGLETON
"Should there be one shared instance?"

ADAPTER
"How can these incompatible interfaces work together?"

DECORATOR
"How can I add behavior without changing the original?"

FACADE
"How can I simplify a complex subsystem?"

STRATEGY
"Which algorithm/behavior should I use?"

OBSERVER
"Who should react when something happens?"

COMMAND
"How can I represent an action as an object?"

REPOSITORY
"How can I separate business logic from persistence?"

DEPENDENCY INJECTION
"Who should provide my dependencies?"

REACTOR
"How can I handle many events efficiently?"
```

---

# 31. Final Tech Lead Mental Model

When designing a system, don't start with:

> "Which design pattern should I use?"

Start with:

```text
1. What problem am I solving?
             ↓
2. What responsibilities exist?
             ↓
3. What changes frequently?
             ↓
4. What should remain stable?
             ↓
5. Where is coupling?
             ↓
6. What abstraction is useful?
             ↓
7. Which pattern naturally fits?
             ↓
8. What are the trade-offs?
             ↓
9. Is the complexity justified?
```

Then select the pattern.

---

# 32. The Most Important Patterns for Your Interviews

For a **10 YOE Full-Stack Developer / Tech Lead / AI Lead / Solution Architect**, prioritize these first:

### 🔥 Tier 1 — Must Know

1. **Strategy**
2. **Factory**
3. **Adapter**
4. **Observer**
5. **Dependency Injection**
6. **Repository**

### 🔥 Tier 2 — Strongly Recommended

7. **Decorator**
8. **Facade**
9. **Builder**
10. **Command**

### 🔥 Tier 3 — Architecture/Advanced Discussion

11. **Singleton**
12. **Reactor**

The important thing is not memorizing 12 definitions.

You should be able to explain:

```text
Problem
   ↓
Design decision
   ↓
Pattern
   ↓
Implementation
   ↓
Trade-off
   ↓
Production use case
   ↓
Alternative
```

That is the difference between a **developer-level answer** and a **Tech Lead / Solution Architect answer**.