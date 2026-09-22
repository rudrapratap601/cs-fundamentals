# 3. SOLID and Object Relationships

[Index](README.md) · [Previous](02_polymorphism_and_language_behavior.md) · [Next](04_patterns_and_design_exercises.md)

## Relationships — core

| Relationship | Meaning | Example |
|---|---|---|
| Dependency | Uses another type temporarily or through an operation | Report generator calls a formatter |
| Association | Objects are connected | Teacher teaches Student |
| Aggregation | Shared whole/part relationship with independent part lifetime | Team references Players |
| Composition | Strong ownership; part belongs to the whole's conceptual lifecycle | Order owns its OrderLines |
| Inheritance | Subtype relationship | Circle implements a Shape contract |

Aggregation/composition are modeling concepts, not automatic garbage-collection instructions. Whether a relationship fits depends on the domain. A database can preserve historical parts after a whole is archived, for example.

## Composition over inheritance

Inheritance is an “is-a” relationship justified by behavioral substitutability. Composition is a “has-a/uses-a” relationship: an object delegates work to collaborators.

**Example:** Instead of creating `EmailDiscountOrder`, `SmsDiscountOrder`, and every other combination, let an OrderService use a DiscountPolicy and a MessageSender. Independent behaviors can vary independently.

Composition adds delegation and object wiring. It is not an absolute ban on inheritance; inheritance can fit stable, coherent subtype contracts.

## Cohesion and coupling

- **High cohesion:** a component's responsibilities belong together.
- **Low coupling:** components depend on fewer details of one another.

Prefer changes that stay local to the responsibility they affect. Splitting every method into a class can reduce readability without improving either property.

## SOLID with concrete examples

### S — Single Responsibility Principle

A module should have one coherent responsibility/reason to change for a stakeholder or policy.

**Problem:** Invoice calculates totals, renders PDFs, and sends email. Pricing, document layout, and delivery rules change independently.

**Improvement:** Keep invoice-related rules in Invoice or a pricing collaborator; use separate rendering and delivery components. SRP does not mean one method per class.

### O — Open/Closed Principle

Design stable parts so new behavior can be added through intended extension points without repeatedly editing those stable parts.

**Problem:** Checkout contains an expanding branch for every discount strategy.

**Improvement:** Depend on a DiscountPolicy and provide implementations. A new policy adds one implementation rather than changing checkout's orchestration. Do not invent extension points for every hypothetical future requirement.

### L — Liskov Substitution Principle

Subtypes should be usable where their parent contract is expected without breaking correctness. They should not strengthen required preconditions, weaken promised postconditions, or violate invariants.

**Problem:** A Bird base type promises `fly()`, but Penguin cannot honor it. Throwing an unsupported-operation exception violates the promise if callers were guaranteed flight.

**Improvement:** Separate Bird from a Flyable capability; only suitable types implement Flyable.

**Classic follow-up:** A mutable Rectangle API independently sets width and height. A Square subtype that forces both equal can break clients expecting independent setters. Mathematical set membership does not guarantee valid behavioral subtyping for a particular API.

### I — Interface Segregation Principle

Clients should not depend on operations they do not need.

**Problem:** A Worker interface forces every implementation to `work()`, `eat()`, and `sleep()`, even robots.

**Improvement:** Expose appropriate focused capabilities. Split by meaningful client roles, not an arbitrary rule that every interface must have one method.

### D — Dependency Inversion Principle

High-level policy and low-level mechanisms should depend on suitable abstractions; the policy should not be tied to a replaceable mechanism's details.

```java
interface OrderRepository {
    void save(String orderId);
}

final class OrderService {
    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = java.util.Objects.requireNonNull(repository);
    }

    public void placeOrder(String orderId) {
        if (orderId == null || orderId.isBlank()) {
            throw new IllegalArgumentException("Missing order ID");
        }
        repository.save(orderId);
    }
}
```

This illustrative Java 11+ snippet lets a database adapter or an in-memory fake implement the repository. It omits real order rules and transaction boundaries to focus on dependencies.

**Dependency injection** supplies dependencies from outside, often through constructors. **Dependency inversion** describes the direction and abstraction of dependencies. Injection can help implement inversion, but they are not synonyms; injecting a concrete database client still couples the consumer to that concrete type.

## Designing contracts

Specify accepted input, output, side effects, failure semantics, and concurrency/lifetime requirements. For example, `save()` must say whether return means “buffered,” “persisted,” or “transaction committed” if callers depend on that distinction.

Avoid exposing mutable internals or long navigation chains such as `order.customer().account().settings().policy().calculate()`. Ask the responsible collaborator for the operation needed, while avoiding meaningless forwarding methods that merely hide every field.

## Interview answers

**Why favor constructor injection?** Dependencies are explicit, required collaborators can be validated immediately, and tests can supply substitutes. Very large constructors may indicate too many responsibilities.

**Does every class need an interface?** No. Add one when it represents a meaningful contract, boundary, or variation—not to satisfy a class-count convention.

**How do you evaluate design quality?** Use concrete changes: adding a delivery channel, changing storage, or adjusting a pricing rule. Examine which modules must change and whether invariants remain protected.
