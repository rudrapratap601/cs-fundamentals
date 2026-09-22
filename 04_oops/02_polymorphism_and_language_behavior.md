# 2. Polymorphism and Language Behavior

[Index](README.md) · [Previous](01_objects_and_four_pillars.md) · [Next](03_solid_and_relationships.md)

## Overloading versus overriding — core

| Aspect | Overloading in Java | Overriding in Java |
|---|---|---|
| Relationship | Same name, different parameter lists | Subclass implementation of inherited instance method |
| Selection | Compile time using declared types and applicable signatures | Runtime dispatch to actual object's implementation |
| Return type alone | Insufficient to distinguish overloads | Compatible/covariant return required |
| Inheritance required | No | Yes, including implementing interface contracts |

An override cannot reduce visibility or broaden checked exceptions beyond the inherited contract. Private methods are not overridden; final methods cannot be overridden; static methods are hidden rather than dynamically overridden.

### Dispatch example

```java
class Animal {
    public String sound() { return "animal"; }
}

class Dog extends Animal {
    @Override
    public String sound() { return "woof"; }
}

class Demo {
    static String label(Animal value) { return "Animal overload"; }
    static String label(Dog value) { return "Dog overload"; }

    public static void main(String[] args) {
        Animal pet = new Dog();
        System.out.println(pet.sound()); // woof: runtime object is Dog
        System.out.println(label(pet));  // Animal overload: declared type is Animal
    }
}
```

Fields are not dynamically dispatched like instance methods. A hidden field accessed through a base-typed reference is selected using the declared reference type. Avoid field hiding because it makes code harder to reason about.

## Interface versus abstract class

| Interface in Java | Abstract class in Java |
|---|---|
| Defines a capability/contract | Can share state and implementation |
| A class can implement multiple interfaces | A class extends only one class |
| No per-instance fields or constructors | Can have instance fields and constructors |
| Can include abstract, default, static, and private methods under applicable language versions | Can contain abstract and concrete methods with access controls |
| Fields are implicitly public static final | Fields can have ordinary instance semantics |

Use an interface for replaceable capabilities; use an abstract class when shared implementation/state and a coherent hierarchy justify it. “Interfaces contain no implementation” is not generally true for Java. Default-method conflicts from unrelated interfaces may require an explicit overriding resolution.

## Casting and type checks

Upcasting a subtype to a supertype preserves the same object. Downcasting requires that the actual object be compatible; otherwise Java throws `ClassCastException`. A cast does not transform an object into another kind.

Frequent type tests and downcasts can suggest missing polymorphic behavior, but occasional boundary checks are reasonable. Do not add inheritance purely to eliminate every conditional.

## Equality, hashing, and identity — core

For Java references, `==` tests object identity. `equals()` can define logical equality. Unless overridden, Object's equality is identity-based.

Equality should be reflexive, symmetric, transitive, consistent, and false against null. Equal objects must have equal hash codes; unequal objects may share a hash code. If you override equals, also implement a compatible hashCode.

```java
import java.util.Objects;

final class UserId {
    private final String value;

    UserId(String value) {
        this.value = Objects.requireNonNull(value);
    }

    @Override
    public boolean equals(Object other) {
        if (this == other) return true;
        if (!(other instanceof UserId)) return false;
        UserId that = (UserId) other;
        return value.equals(that.value);
    }

    @Override
    public int hashCode() {
        return value.hashCode();
    }
}
```

Do not mutate fields used in hash-based key equality while the key is stored in a hash map/set: lookup may search a different bucket afterward. Using immutable value objects helps preserve the contract.

## Immutability, copying, and parameter passing

An immutable object cannot observably change after construction. Helpful techniques: private final fields, no mutators, defensive copying of mutable inputs, and safe return values. `final` on a reference prevents reassignment, not mutation of the referenced object. An unmodifiable view is not necessarily a snapshot; the underlying collection may still change elsewhere.

- **Shallow copy:** copies the outer object but shares referenced nested objects.
- **Deep copy:** copies relevant nested state according to a defined ownership model; cycles/shared identity make this nontrivial.
- **Java is pass-by-value:** a parameter receives a copy of a primitive value or reference value. Mutating the referenced object can be visible to the caller; reassigning the parameter does not reassign the caller's variable.

## Lifetime and language-specific differences

Java garbage collection reclaims unreachable memory; it does not guarantee timely release of files, sockets, or locks. Use explicit resource management such as try-with-resources for AutoCloseable resources.

| Topic | Java | C++ distinction |
|---|---|---|
| Class inheritance | One superclass, multiple interfaces | Multiple base classes supported |
| Method dispatch | Overridable instance methods dispatch dynamically | Runtime overriding uses virtual methods |
| Lifetime | Garbage-collected memory | RAII and deterministic destruction for scoped objects |
| Copying | Assigning references shares the object | Value copies and copy/move semantics are central |
| Base destruction | No C++-style destructor | Deleting a derived object through a base pointer generally requires a virtual base destructor |

In C++, copying a derived object into a base object by value can cause **object slicing**. The **diamond problem** concerns repeated inheritance paths; virtual inheritance can share a common base subobject, with added complexity. Do not transfer these mechanics directly to Java.

## Interview answers

**Can Java overload only by return type?** No; return type alone does not give a distinct overload signature.

**Does final imply immutable?** No. A final collection reference can still refer to a mutable collection.

**Does garbage collection prevent memory leaks?** No. Objects still reachable through unnecessary references can retain memory indefinitely.
