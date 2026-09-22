# 4. Patterns and Design Exercises

[Index](README.md) · [Previous](03_solid_and_relationships.md) · [Next](05_interview_practice.md)

## Patterns solve recurring problems

A design pattern names a reusable structure and its trade-offs. Start with the problem; do not force a pattern into every class.

| Pattern | Problem it solves | Cost or caution |
|---|---|---|
| Factory | Centralize/abstract creation decisions | Can add indirection without useful variation |
| Builder | Construct objects with many meaningful options/steps | Must still validate the final object |
| Strategy | Replace an algorithm behind one contract | Extra collaborators and configuration |
| Observer | Notify interested parties of changes | Ordering, failures, unsubscribe, and lifetime need care |
| Adapter | Make an existing API fit a required interface | Must reconcile semantics, not just method names |
| Decorator | Add behavior by wrapping the same interface | Wrapper ordering and debugging complexity |
| Facade | Provide a simpler entry point to a subsystem | Can become overly broad |
| State | Vary behavior with an object's state | Additional types may be excessive for a tiny state machine |
| Singleton | Restrict an instance within a defined scope | Global state, hidden dependencies, test/concurrency costs |

Factory Method uses an overridable creation operation; Abstract Factory produces related object families. A plain conditional factory function is useful but is not automatically both named patterns.

## Worked Strategy example

```java
interface DiscountPolicy {
    long discountInCents(long subtotalInCents);
}

final class NoDiscount implements DiscountPolicy {
    @Override
    public long discountInCents(long subtotalInCents) {
        return 0;
    }
}

final class TenPercentDiscount implements DiscountPolicy {
    @Override
    public long discountInCents(long subtotalInCents) {
        return subtotalInCents / 10; // deliberate rounding down in this example
    }
}

final class Checkout {
    private final DiscountPolicy discountPolicy;

    Checkout(DiscountPolicy discountPolicy) {
        this.discountPolicy = java.util.Objects.requireNonNull(discountPolicy);
    }

    public long totalInCents(long subtotalInCents) {
        if (subtotalInCents < 0) {
            throw new IllegalArgumentException("Negative subtotal");
        }
        long discount = discountPolicy.discountInCents(subtotalInCents);
        if (discount < 0 || discount > subtotalInCents) {
            throw new IllegalStateException("Invalid discount");
        }
        return subtotalInCents - discount;
    }
}
```

`new Checkout(new TenPercentDiscount()).totalInCents(1999)` returns `1800`: the discount is 199 cents. State rounding requirements explicitly; another product might require a different policy. The policy contract assumes a nonnegative subtotal and a discount within that subtotal, with Checkout validating its boundary.

The stable checkout operation does not need a branch for every new discount type. For a one-off fixed formula, this structure may be unnecessary.

## Design exercise: notification service

**Requirements:** Send a message using email or SMS; allow another channel later; test application behavior without real network delivery.

**Model:**

```text
NotificationService → MessageSender
                       ├─ EmailSender → email provider adapter
                       └─ SmsSender   → SMS provider adapter
```

1. Define MessageSender's recipient format, message constraints, and failure contract.
2. Inject an implementation into NotificationService; use a fake sender in tests.
3. Use a factory or composition root to choose the channel when needed.
4. Add a provider adapter if its API does not match the application contract.
5. Add logging/metrics wrappers when useful, preserving the send contract.

**Follow-ups:** Retrying can duplicate delivery. Decide which failures are retryable and whether the provider supports idempotency. Asynchronous queuing changes what success means: “accepted for processing” is different from “delivered.” OOP boundaries do not solve delivery guarantees by themselves.

## Design exercise: parking lot

First clarify vehicle categories, spot compatibility, allocation policy, pricing, capacity, and concurrent entry/exit. Do not immediately create an inheritance hierarchy for every noun.

| Type | Responsibility |
|---|---|
| Vehicle | Identity and relevant category |
| ParkingSpot | Identity, compatibility, occupancy |
| ParkingTicket | Entry time, assigned spot, lifecycle status |
| SpotAllocationPolicy | Select a compatible available spot |
| PricingPolicy | Compute a fee from well-defined inputs |
| ParkingLotService | Coordinate entry/exit and repository operations |

**Entry flow:** Validate request → atomically reserve compatible spot → create ticket → return ticket.

**Exit flow:** Validate active ticket → compute fee → coordinate payment/exit rules → release spot and close ticket consistently.

**Invariants:** One active occupant per spot; one active assignment per ticket; exiting twice must not release an unrelated occupant's spot. Concurrency needs locking or transactional coordination around selection/reservation. A read of “available” followed by an unprotected write is insufficient.

**Trade-off:** Strategies help when allocation/pricing really vary. A small fixed lot may need fewer types. External payments can fail independently from local updates; clarify workflow and recovery rather than claiming one in-memory method makes them atomic.

## Common pattern comparisons

**Strategy versus State:** Strategy represents a chosen algorithm; State represents behavior associated with lifecycle state and transitions. Structures can look similar, but intent differs.

**Adapter versus Decorator:** Adapter changes the interface to fit a caller; Decorator preserves an interface while adding behavior.

**Factory versus dependency injection:** A factory creates/selects objects; injection supplies collaborators to consumers. They can work together.

**Singleton versus static utility:** A singleton is an object with instance semantics within a scope; a utility exposes static functions. Neither justifies hidden mutable global state. “Only one in the whole distributed system” is not guaranteed by a language-level singleton.
