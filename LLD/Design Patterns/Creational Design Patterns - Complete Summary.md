# Creational Design Patterns - Complete Summary

## Overview
Creational design patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. They help make a system independent of how its objects are created, composed, and represented.

---

## 1. Singleton Pattern

### Purpose
Ensures a class has only one instance and provides a global point of access to it.

### State Behavior
- **Mutable State**: The singleton instance maintains state across the application
- **Returns**: Always returns the **same instance** (not a new object)
- **Thread Safety**: Requires careful implementation in multi-threaded environments

### Example Use Cases
- Database connections
- Logger instances
- Configuration managers
- Cache managers

### Key Characteristics
```
getInstance() → Returns existing instance
State: Shared across entire application
Memory: One object throughout application lifecycle
```

---

## 2. Factory Method Pattern

### Purpose
Defines an interface for creating an object, but lets subclasses decide which class to instantiate.

### State Behavior
- **Returns**: Creates and returns a **new object** each time
- **State**: Each created object has independent state
- **Polymorphism**: Returns different types based on subclass implementation

### Example Use Cases
- Document creators (WordDocument, PDFDocument)
- GUI element factories (WindowsButton, MacButton)
- Payment processors (CreditCardPayment, PayPalPayment)

### Key Characteristics
```
createProduct() → Returns new instance every call
State: Independent for each created object
Flexibility: Subclasses determine concrete type
```

---

## 4. Builder Pattern

### Purpose
Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

### State Behavior
- **Returns**: Creates and returns a **new object** after building
- **State**: Builder accumulates state during construction, final object is immutable (usually)
- **Flexibility**: Same builder can create different configurations

### Example Use Cases
- Complex object construction (House, Car, Computer)
- Query builders (SQL, MongoDB queries)
- Document builders (HTML, XML)
- Immutable objects with many optional parameters

### Key Characteristics
```
Builder.setProperty() → Returns builder (for chaining)
Builder.build() → Returns new constructed object
State: Builder is mutable during construction
Final Object: Often immutable after construction
```

---

## 5. Prototype Pattern

### Purpose
Creates new objects by copying an existing object (prototype), avoiding expensive creation operations.

### State Behavior
- **Returns**: Creates and returns a **new object** (clone of prototype)
- **State**: New object is a copy; can be shallow or deep copy
- **Performance**: Faster than creating from scratch for complex objects

### Example Use Cases
- Cloning complex objects
- Game object spawning (enemies, items)
- Document templates
- Object pools

### Key Characteristics
```
clone() → Returns new object (copy of prototype)
State: New object initially has same state as prototype
Independence: Cloned object can be modified independently
Copy Type: Shallow vs Deep copy consideration important
```

---

## Comparison Table

| Pattern | Returns New Object? | State Sharing | Primary Use Case |
|---------|---------------------|---------------|------------------|
| **Singleton** | ❌ No (same instance) | ✅ Shared state | Single instance needed |
| **Factory Method** | ✅ Yes | ❌ Independent | Defer instantiation to subclasses |
| **Builder** | ✅ Yes | ❌ Independent | Complex object construction |
| **Prototype** | ✅ Yes (clone) | ❌ Independent (copy) | Clone existing objects |

---

## State Availability Summary

### Patterns with Shared State
- **Singleton**: Single instance, state shared across application

### Patterns with Independent State
- **Factory Method**: Each product has own state
- **Builder**: Each built object has own state
- **Prototype**: Each clone has own state (initially copied)

---

## Object Creation Summary

### Always Returns Same Instance
- **Singleton** - `getInstance()` returns the single instance

### Always Returns New Instance
- **Factory Method** - `createProduct()` returns new object
- **Builder** - `build()` returns new constructed object
- **Prototype** - `clone()` returns new copied object

---

## When to Use What?

### Use Singleton when:
- Exactly one instance needed
- Global access point required
- Shared state across application

### Use Factory Method when:
- Class can't anticipate type of objects to create
- Subclasses should specify objects they create
- Localize object creation logic

### Use Builder when:
- Object construction is complex
- Many optional parameters
- Want immutable objects
- Step-by-step construction needed

### Use Prototype when:
- Object creation is expensive
- Similar objects with slight variations needed
- Avoid subclassing
- Dynamic object creation

---

## Best Practices

1. **Singleton**: Use dependency injection instead when possible; ensure thread-safety
2. **Factory Method**: Keep creation logic simple and focused
3. **Builder**: Make built objects immutable; use fluent interface
4. **Prototype**: Implement deep copy when objects have complex references

---

## Common Pitfalls

- **Singleton**: Can make testing difficult, creates hidden dependencies
- **Factory Method**: Can lead to parallel class hierarchies
- **Builder**: Can be overkill for simple objects
- **Prototype**: Clone method must handle all object state correctly

---

*This summary covers the four main creational design patterns with focus on state management and object creation behavior.*
