# 5. OOP Interview Practice

[Index](README.md) · [Previous](04_patterns_and_design_exercises.md)

## Rapid recall

| Question | Answer checkpoint |
|---|---|
| Four pillars? | Encapsulation, abstraction, inheritance, polymorphism |
| Abstraction versus encapsulation? | Relevant public capability versus controlled implementation/state boundary |
| Overloading versus overriding? | Compile-time signature selection versus runtime instance-method dispatch |
| Interface versus abstract class? | Capability/multiple contracts versus shared state/implementation hierarchy |
| Composition versus inheritance? | Delegation/ownership versus behavioral subtype relationship |
| Does final imply immutable? | No; referenced objects can remain mutable |
| Java parameter passing? | Always by value, including copied reference values |
| equals/hashCode contract? | Equal objects must have equal hashes; unequal objects may collide |
| DI versus DIP? | Supplying dependencies versus abstract dependency direction |

## Predict and explain

### 1. Static type versus runtime type

```java
Animal a = new Dog();
System.out.println(a.sound());
System.out.println(Demo.label(a));
```

Using the classes in [chapter 2](02_polymorphism_and_language_behavior.md), output is `woof`, then `Animal overload`. Runtime type controls overridden instance behavior; declared type participates in overload selection.

### 2. Reference reassignment

```java
static void replace(StringBuilder value) {
    value = new StringBuilder("new");
}

static void append(StringBuilder value) {
    value.append("!");
}
```

If a caller's builder starts as `old`, calling replace does not change the caller's reference or builder. Calling append mutates that shared builder to `old!`. Both methods receive a reference value by value.

### 3. Mutable hash-map key

**Question:** A key's name determines equals/hashCode. You insert it, change its name, then cannot find it. Why?

**Answer:** The new hash may point to a different bucket from the insertion location. Preserve hash/equality-relevant state while stored, commonly using immutable keys.

### 4. Public list getter

**Question:** A class has a private final list but returns it directly. Is the class immutable?

**Answer:** Not if callers can mutate that list. Protect mutable inputs and outputs; distinguish snapshots from unmodifiable views and consider mutable elements inside the collection too.

## Design questions with answer outlines

**Apply LSP to Rectangle/Square.** Examine the API contract, not only geometry. If independent dimension setters are promised, forcing dimensions equal can break clients. Prefer a different abstraction, such as an immutable shape with area behavior, when that matches requirements.

**Add SMS to an email-only service.** Define a suitable sender contract, inject the implementation, and isolate provider differences. Clarify failure/recipient semantics; do not merely rename an email-specific API to a generic name.

**Choose inheritance or composition for discounts.** Different discount policies are collaborators used by checkout. Composition with Strategy avoids a checkout subclass for every discount combination.

**Design a library loan system.** Distinguish BookTitle from a physical BookCopy, plus Member and Loan. Protect one-active-loan-per-copy and applicable member limits. Borrow/return coordination belongs in a service with transactional persistence, not a public setter on availability.

**Make a class testable.** Expose dependencies such as storage, clock, or sender through suitable contracts where needed. Test observable behavior and invariants; avoid tests tied to incidental private method structure.

## A five-step design answer

1. Clarify requirements and scope.
2. Identify entities/value objects and invariants.
3. Assign responsibilities and specify contracts.
4. Walk through a success path and important failures/concurrent actions.
5. Explain one likely extension and the trade-offs of the chosen design.

## Mistakes to eliminate

- “Inheritance is always the best way to reuse code.”
- “Java passes objects by reference.”
- “Static methods override dynamically.”
- “Private fields plus setters guarantee encapsulation.”
- “Garbage collection closes all resources promptly.”
- “SOLID requires an interface for every class.”
- “A design pattern is good simply because it has a name.”
