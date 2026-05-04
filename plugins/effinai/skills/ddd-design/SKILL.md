---
name: ddd-design
description: "Domain-Driven Design patterns including bounded contexts, aggregates, and context mapping"
version: "1.0.0"
tags: [architecture, ddd, design-patterns, domain-modeling]
---

# Domain-Driven Design

Apply DDD strategic and tactical patterns to model complex business domains in code.

## Strategic Patterns

### Bounded Contexts

A bounded context is a boundary within which a domain model is consistent and a ubiquitous language applies.

**Identification steps**:
1. List all domain concepts (nouns from requirements)
2. Find concepts that have different meanings in different areas (e.g., "Account" in billing vs. authentication)
3. Draw boundaries where the same word means different things
4. Each boundary is a bounded context with its own model

**Rules**:
- One team owns one bounded context
- Models do NOT leak across boundaries
- Communication between contexts uses explicit contracts (APIs, events)
- Each context has its own database schema (no shared tables)

### Context Map

Document relationships between bounded contexts:

| Relationship | Description | When to Use |
|-------------|-------------|-------------|
| **Shared Kernel** | Two contexts share a small common model | Tightly coupled teams that coordinate releases |
| **Customer-Supplier** | Upstream context serves downstream | Downstream depends on upstream's API |
| **Conformist** | Downstream adopts upstream's model as-is | No leverage to influence upstream |
| **Anti-Corruption Layer** | Downstream translates upstream's model | Upstream model is legacy or misaligned |
| **Open Host Service** | Upstream provides a published API | Multiple downstream consumers |
| **Published Language** | Shared schema (protobuf, JSON Schema) | Cross-org integration |
| **Separate Ways** | No integration; contexts are independent | Cost of integration exceeds benefit |

### Anti-Corruption Layer (ACL)

Translate between external models and your domain model:

```typescript
// ACL translates external payment provider model to our domain
class PaymentACL {
  constructor(private externalClient: StripeClient) {}

  async chargeCustomer(order: Order): Promise<PaymentResult> {
    // Translate our domain model to Stripe's model
    const stripeCharge = {
      amount: order.total.cents,
      currency: order.total.currency.code,
      customer: order.customer.externalPaymentId,
      metadata: { orderId: order.id.value }
    };
    const result = await this.externalClient.charges.create(stripeCharge);
    // Translate Stripe's response back to our domain
    return PaymentResult.fromExternal(result.id, result.status);
  }
}
```

## Tactical Patterns

### Entities

Objects with identity that persists across state changes.

```typescript
class Order {
  constructor(
    public readonly id: OrderId,       // Identity
    private items: OrderItem[],
    private status: OrderStatus
  ) {}

  addItem(product: ProductId, quantity: number, price: Money): void {
    if (this.status !== OrderStatus.DRAFT) {
      throw new OrderNotEditableError(this.id);
    }
    this.items.push(new OrderItem(product, quantity, price));
  }

  get total(): Money {
    return this.items.reduce((sum, item) => sum.add(item.subtotal), Money.zero());
  }
}
```

### Value Objects

Immutable objects defined by their attributes, not identity.

```typescript
class Money {
  private constructor(
    public readonly cents: number,
    public readonly currency: Currency
  ) {
    if (cents < 0) throw new InvalidMoneyError("Amount cannot be negative");
  }

  static of(amount: number, currency: Currency): Money {
    return new Money(Math.round(amount * 100), currency);
  }

  add(other: Money): Money {
    if (!this.currency.equals(other.currency)) {
      throw new CurrencyMismatchError(this.currency, other.currency);
    }
    return new Money(this.cents + other.cents, this.currency);
  }

  equals(other: Money): boolean {
    return this.cents === other.cents && this.currency.equals(other.currency);
  }
}
```

### Aggregates

Cluster of entities and value objects with a single root entity that enforces invariants.

**Rules**:
- Only the aggregate root is accessible from outside
- All modifications go through the root
- One transaction = one aggregate
- Reference other aggregates by ID only, never by direct object reference
- Keep aggregates small (prefer more small aggregates over fewer large ones)

```typescript
class Order {  // Aggregate Root
  private items: OrderItem[];  // Internal entity
  // OrderItem is NOT accessible from outside Order

  submit(): DomainEvent[] {
    if (this.items.length === 0) throw new EmptyOrderError();
    if (this.total.cents > this.customer.creditLimit.cents) {
      throw new CreditLimitExceededError();
    }
    this.status = OrderStatus.SUBMITTED;
    return [new OrderSubmitted(this.id, this.total, new Date())];
  }
}
```

### Domain Events

Record that something meaningful happened in the domain.

```typescript
interface DomainEvent {
  readonly eventId: string;
  readonly occurredAt: Date;
  readonly aggregateId: string;
  readonly eventType: string;
}

class OrderSubmitted implements DomainEvent {
  readonly eventType = "order.submitted";
  constructor(
    public readonly aggregateId: string,
    public readonly total: Money,
    public readonly occurredAt: Date,
    public readonly eventId = crypto.randomUUID()
  ) {}
}
```

**Event handling**: Use domain events for cross-aggregate side effects. When an Order is submitted, an event triggers inventory reservation in a separate bounded context.

### Repositories

Abstraction over persistence for aggregates.

```typescript
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
  nextId(): OrderId;
}
// Implementation details (SQL, Mongo, etc.) live in infrastructure layer
```

### Domain Services

Operations that don't naturally belong to any entity or value object.

```typescript
class PricingService {
  calculateDiscount(customer: Customer, items: OrderItem[]): Money {
    // Business logic that spans multiple aggregates
    const loyaltyDiscount = customer.loyaltyTier.discountRate;
    const volumeDiscount = items.length > 10 ? 0.05 : 0;
    const rate = Math.min(loyaltyDiscount + volumeDiscount, 0.30);
    return items.reduce((t, i) => t.add(i.subtotal), Money.zero()).multiply(rate);
  }
}
```

## Layer Architecture

```
Presentation  -->  Application  -->  Domain  <--  Infrastructure
(Controllers)    (Use Cases)     (Entities,      (Repositories,
                 (Commands,       Value Objects,   External APIs,
                  Queries)        Domain Events,   Persistence)
                                  Services)
```

**Dependency rule**: Domain has ZERO dependencies on other layers. Infrastructure depends on Domain (implements repository interfaces).

## Modeling Checklist

- [ ] Ubiquitous language documented and used consistently in code
- [ ] Bounded contexts identified with clear boundaries
- [ ] Context map shows all relationships between contexts
- [ ] Aggregates enforce all business invariants
- [ ] Value objects are immutable with equality by value
- [ ] Domain events capture all meaningful state transitions
- [ ] Anti-corruption layers isolate external models
- [ ] No domain logic in application or infrastructure layers

## Agent Assignments

| Agent | Role |
|-------|------|
| `system-architect` | Define bounded contexts and context map |
| `coder` | Implement tactical patterns |
| `reviewer` | Validate DDD rule compliance |
| `researcher` | Domain knowledge extraction |
