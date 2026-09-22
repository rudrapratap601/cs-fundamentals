# 1. Objects and the Four Pillars

[Index](README.md) · [Next](02_polymorphism_and_language_behavior.md)

## Class and object

A class defines a type's structure and behavior in class-based OOP. An object is an instance with identity, state, and behavior. OOP also exists in forms other than class-based systems, such as prototype-based programming.

- **Identity:** which object this is, even when another object has equal values.
- **State:** current data held or referenced by the object.
- **Behavior:** operations available to clients.
- **Invariant:** a rule that should hold for valid observable states.

## Encapsulation — core

Encapsulation groups state and behavior behind a controlled boundary. Information hiding prevents callers from depending on changeable internal details. Making fields private is a useful mechanism but does not automatically create a good boundary.

```java
final class BankAccount {
    private long balanceInCents;

    BankAccount(long openingBalanceInCents) {
        if (openingBalanceInCents < 0) {
            throw new IllegalArgumentException("Negative opening balance");
        }
        balanceInCents = openingBalanceInCents;
    }

    public void deposit(long amountInCents) {
        if (amountInCents <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balanceInCents = Math.addExact(balanceInCents, amountInCents);
    }

    public void withdraw(long amountInCents) {
        if (amountInCents <= 0 || amountInCents > balanceInCents) {
            throw new IllegalArgumentException("Invalid withdrawal");
        }
        balanceInCents -= amountInCents;
    }

    public long balanceInCents() {
        return balanceInCents;
    }
}
```

The object protects nonnegative balance by exposing meaningful operations rather than an unrestricted setter. Integer minor units avoid binary floating-point rounding in this simplified single-currency model. Overflow is checked for deposits. This teaching class is not thread-safe and does not implement transfers, persistence, or a complete monetary domain.

**Trap:** A getter returning a mutable internal list can leak state even if the field itself is private.

## Abstraction — core

Abstraction presents the capabilities relevant to a caller while hiding implementation details. A caller can ask a storage service to save a document without knowing the storage engine's page layout.

```java
interface MessageSender {
    void send(String recipient, String message);
}
```

The interface exposes a capability, but its contract also needs behavioral meaning: allowed inputs, failure behavior, and what “send succeeded” promises. An interface alone does not document every semantic requirement.

**Abstraction versus encapsulation:** Abstraction chooses what the caller sees; encapsulation controls access to implementation/state. They support each other but are not synonyms.

## Inheritance — core

Inheritance derives a type from another, potentially reusing behavior and establishing a subtype relationship. Use it when the subtype can honor the parent's contract, not merely because some fields look similar.

```java
abstract class Shape {
    public abstract double area();
}

final class Circle extends Shape {
    private final double radius;

    Circle(double radius) {
        if (!Double.isFinite(radius) || radius < 0) {
            throw new IllegalArgumentException("Invalid radius");
        }
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

This is a simplified geometric model; floating-point overflow/precision limits still apply to area. Inheritance can couple subclasses to parent implementation decisions. Prefer shallow, justified hierarchies over deep trees created for code reuse alone.

## Polymorphism — core

Polymorphism lets one interface or operation work with multiple forms. In subtype polymorphism, callers use a base contract while the runtime selects an implementation.

```java
Shape shape = new Circle(2.0);
System.out.println(shape.area()); // approximately 12.566
```

The caller knows Shape; the runtime invokes Circle's area. Overloading and generics are also discussed as forms of polymorphism, but their mechanisms differ from runtime method overriding.

## Constructors and access

A constructor establishes initial state. In Java, constructors are not inherited or overridden, though they can be overloaded. A compiler-provided no-argument constructor is supplied only when no constructor is declared; it is not automatically supplied alongside your own constructors.

| Java access | Main visibility |
|---|---|
| `private` | Declaring class/nested access rules |
| No modifier | Same package |
| `protected` | Same package and subclasses, with extra cross-package access restrictions |
| `public` | Accessible where the containing type/module is accessible |

Do not equate Java `protected` with unrestricted access through any superclass reference from any subclass package.

## Interview answers

**Are getters/setters encapsulation?** They can support it, but unrestricted setters can expose invalid state transitions. Prefer domain operations when they preserve invariants.

**Can you have abstraction without an abstract class?** Yes. Functions, modules, interfaces, and ordinary classes can all expose useful abstractions.

**Is inheritance necessary for all OOP?** No. Objects and composition can provide effective object-oriented designs with little implementation inheritance.
