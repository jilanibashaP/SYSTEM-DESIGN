# 📚 Creational Design Patterns

---

## 📖 Table of Contents

1. [Introduction to Design Patterns](#1-introduction-to-design-patterns)
2. [Three Types of Design Patterns](#2-three-types-of-design-patterns)
3. [Creational Patterns Overview](#3-creational-patterns-overview)
4. [The Four Core Questions](#4-the-four-core-questions)
5. [Singleton Pattern](#5-singleton-pattern)
6. [Builder Pattern](#6-builder-pattern)
7. [Prototype Pattern](#7-prototype-pattern)
8. [Factory Pattern (Brief)](#8-factory-pattern-brief)
9. [Static vs Non-Static](#9-static-vs-non-static)
10. [Comparison Tables](#10-comparison-tables)
11. [Interview Guide](#11-interview-guide)

---

## 1. Introduction to Design Patterns

### 🎯 What are SOLID Principles & Design Patterns?

**Simple Explanation:**

| Concept | Meaning |
|---------|---------|
| **SOLID Principles** | Rules for writing good, clean, flexible code |
| **Design Patterns** | Ready-made solutions that follow these rules |

**One-Liner:**
```
SOLID = Rules for good design
Design Patterns = Practical examples of those rules
```

### 🏠 Real-Life Analogy

Think of building a house:
- **SOLID principles** = Building codes and safety regulations
- **Design patterns** = Proven architectural blueprints that follow those codes

### 💡 Quick Example

**❌ Bad Code (Violates SOLID):**
```java
class PaymentProcessor {
    void processPayment(String type) {
        if (type.equals("credit")) {
            // credit card logic
        } else if (type.equals("paypal")) {
            // paypal logic
        }
        // Hard to extend, violates Open-Closed Principle
    }
}
```

**✅ Good Code (Uses Design Pattern):**
```java
interface Payment {
    void process();
}

class CreditCardPayment implements Payment {
    public void process() { /* credit logic */ }
}

class PayPalPayment implements Payment {
    public void process() { /* paypal logic */ }
}

// Factory Pattern - follows SOLID principles ✓
```

---

## 2. Three Types of Design Patterns

### 📊 Overview

| Type | Focus | Main Question | Examples |
|------|-------|---------------|----------|
| **Creational** | Object creation | "How should I create this object?" | Singleton, Factory, Builder |
| **Structural** | Class structure | "How should I connect these classes?" | Adapter, Decorator, Facade |
| **Behavioral** | Object interaction | "How should objects communicate?" | Strategy, Observer, Command |

### 1️⃣ Creational Design Patterns

**Focus:** How objects are created

**Benefits:**
- ✓ Avoid tight coupling with `new`
- ✓ Make object creation flexible
- ✓ Hide complex creation logic

**Think:** *"How should I create this object?"*

---

### 2️⃣ Structural Design Patterns

**Focus:** How classes and objects are connected

**Benefits:**
- ✓ Combine classes/objects cleanly
- ✓ Make code easier to extend
- ✓ Improve code organization

**Think:** *"How should I connect these classes?"*

---

### 3️⃣ Behavioral Design Patterns

**Focus:** How objects communicate and behave

**Benefits:**
- ✓ Remove complex `if-else` chains
- ✓ Make behavior changeable
- ✓ Improve object interaction

**Think:** *"How should these objects talk to each other?"*

---

### 🎯 Interview Answer

> "Creational patterns deal with object creation, Structural patterns deal with class structure, and Behavioral patterns deal with object interaction."

---

## 3. Creational Patterns Overview

### 🔍 What are Creational Design Patterns?

**Simple Definition:**
> Creational Design Patterns decide **how** and **when** objects should be created.

They control object creation instead of using `new` everywhere.

---

### 🚫 Why NOT use `new` everywhere?

#### Problems with `new`:

| Problem | Impact |
|---------|--------|
| **Tight coupling** | Code directly depends on concrete classes |
| **Hard to change** | Changing one class breaks many others |
| **Violates SOLID** | Breaks Open-Closed and Dependency Inversion principles |

#### Solution - Creational Patterns:

| Benefit | How |
|---------|-----|
| **Loose coupling** | Depend on interfaces, not concrete classes |
| **Centralized creation** | One place controls object creation |
| **Flexible code** | Easy to change implementations |

---

### 🔑 Main Creational Patterns

| Pattern | Simple Meaning | Use Case |
|---------|---------------|----------|
| **Singleton** | Only one object for whole app | Database connection, Logger, Config |
| **Factory** | Let a factory decide which object to create | Payment methods, Notifications |
| **Abstract Factory** | Factory of factories | UI themes, Cross-platform apps |
| **Builder** | Step-by-step object creation | Complex objects with many fields |
| **Prototype** | Clone existing object | Game characters, Templates |

---

### 🎯 Interview Answer

> "Creational design patterns focus on when and how objects should be created, instead of directly using `new`, to make the system flexible and loosely coupled."

---

## 4. The Four Core Questions

Creational patterns answer **4 fundamental questions**:

### 1️⃣ WHEN should an object be created?

**Question:** Decide the right time to create an object

**Strategies:**
- Create only when needed (lazy initialization)
- Create once and reuse (singleton)

**Examples:**
- Database connection → create once, reuse
- Heavy object → create only when required

**Patterns:** Singleton, Lazy initialization

---

### 2️⃣ HOW should we create an object?

**Question:** Decide the process of creation

**Instead of this:**
```java
A a = new A();  // Direct creation
```

**We want:**
- Hide complex creation logic
- Avoid constructors with many parameters
- Decide object type at runtime

**Examples:**
- Builder → step-by-step creation with validation
- Factory → decide which class to create based on input
- Prototype → clone existing object

**Patterns:** Factory, Builder, Abstract Factory, Prototype

---

### 3️⃣ WHERE should we create an object?

**Question:** Decide who is responsible for creating it

> ⭐ **This is the MOST IMPORTANT question!**

#### ❌ Bad Design (Tight Coupling)

```java
class Client {
    A a = new A();   // Client creates object directly
}
```

**Problems:**
- Client knows concrete class
- Tight coupling
- Hard to change later
- Violates Dependency Inversion Principle

**If tomorrow `A` changes to `B` → Client code must change ❌**

#### ✅ Good Design (Loose Coupling)

```java
class Client {
    A a;   // Client only uses object, doesn't create it
}
```

**Creation happens outside:**
- Factory class decides
- Builder constructs it
- Framework (like Spring) injects it

**Golden Rule:**
> **Use objects, don't create them in client code**

Or simply:
> **Client should focus on business logic, not object creation**

---

### 💡 Spring Boot Example

```java
@Service
public class OrderService {
    @Autowired
    PaymentService paymentService;  // Spring creates and injects
}
```

**Spring decides:**
- ✓ **When** to create the object
- ✓ **How** to create the object
- ✓ **Where** to create the object

**You just use it!** ✅

---

### 4️⃣ HOW MANY objects should be created?

**Question:** Decide number of instances

**Options:**
- Only one object? → Singleton
- Multiple objects? → Factory, Prototype
- Clone existing object? → Prototype

**Examples:**
- One config object → Singleton
- Multiple payment processors → Factory
- Copy existing object → Prototype

**Patterns:** Singleton, Prototype

---

### 📊 Quick Summary Table

| Pattern | Main Question | Answer |
|---------|--------------|--------|
| **Singleton** | How many objects? | Only one |
| **Builder** | How to create? | Step by step |
| **Factory** | Where & how to create? | Factory decides |
| **Prototype** | How to create copies? | Clone existing |

---

### 🎯 Interview Answer

> "Creational design patterns answer when, how, where, and how many objects should be created, so that client code stays simple, flexible, and loosely coupled."

---

## 5. Singleton Pattern

### 🤔 What is Singleton?

**Simple Definition:**
> Singleton ensures the system has **exactly ONE** shared instance of a class.

---

### 🚨 The Problem Without Singleton

```java
Config c1 = new Config();
Config c2 = new Config();
```

**Now you have:**
- ❌ Two config objects
- ❌ Different values possible
- ❌ Inconsistent behavior

**Real Problems:**
- Multiple DB connections → performance issue
- Multiple loggers → duplicate logs
- Multiple cache objects → data mismatch

---

### ✅ What Singleton Solves

**Singleton ensures:**
- ✓ Only one object exists
- ✓ Global access to that object
- ✓ Controlled creation

```java
Config config = Config.getInstance();
```

**Everyone gets the same object!** ✅

---

### 🎯 When to Use Singleton?

| Use Case | Why Singleton? |
|----------|---------------|
| **Database connection** | Expensive to create, should be shared |
| **Logger** | One log stream for entire app |
| **Configuration** | Single source of truth |
| **Cache** | Shared memory across app |

---

### ⚠️ When NOT to Use Singleton?

**Avoid Singleton when:**
- ❌ You need multiple instances
- ❌ Object holds request/user-specific data
- ❌ Testing requires mocking (Singletons are hard to mock)

---

### 🔧 How to Create a Singleton?

**3 Rules:**

1. **Private constructor** → Prevents `new Singleton()`
2. **Static instance** → Shared across entire application
3. **Public static method** → Global access point

---

### 📝 Implementation Types

#### 1. Basic Singleton (Lazy Initialization)

```java
class Singleton {
    private static Singleton instance;

    private Singleton() {}  // Private constructor

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Usage:**
```java
Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();

System.out.println(s1 == s2);  // true ✅
```

**⚠️ Problem:** Not thread-safe! Two threads can create two objects.

---

#### 2. Thread-Safe Singleton

**Option A: Synchronized Method**

```java
public static synchronized Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
}
```

- ✓ Thread-safe
- ❌ Slower (locks every call)

**Option B: Double-Checked Locking (Recommended)**

```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {              // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {      // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

- ✓ Thread-safe
- ✓ Fast (locks only when creating)
- ✓ Interview favorite

---

#### 3. Enum Singleton (BEST PRACTICE) ⭐

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing something...");
    }
}
```

**Usage:**
```java
Singleton s1 = Singleton.INSTANCE;
Singleton s2 = Singleton.INSTANCE;

System.out.println(s1 == s2); // true ✅
s1.doSomething();
```

**Why Enum is the Best:**
- ✓ Thread-safe (guaranteed by JVM)
- ✓ Serialization-safe (JVM handles it)
- ✓ Reflection-safe (JVM blocks it)
- ✓ Cleanest code
- 🔥 **Best practice in Java**

---

### 🛡️ Why Enum Singleton is Superior

#### 1. Thread-Safe by Default

- JVM guarantees enum instances are created once
- No need for `synchronized`, `volatile`, or double-check locking

#### 2. Serialization-Safe

**Normal Singleton breaks:**
```java
// Deserializing creates a NEW instance ❌
ObjectInputStream → creates new instance
```

**Enum Singleton:**
- JVM ensures same instance during deserialization ✅
- No need for `readResolve()` method

#### 3. Reflection-Safe

**Normal Singleton can be broken:**
```java
constructor.setAccessible(true);
constructor.newInstance();  // Creates second instance ❌
```

**Enum Singleton:**
```java
// Trying to create via reflection
c.newInstance();  // Throws IllegalArgumentException ✅
// JVM prevents reflective creation of enum instances
```

---

### 💥 Breaking Normal Singleton (Examples)

#### Example 1: Breaking with Reflection

```java
public class ReflectionBreak {
    public static void main(String[] args) throws Exception {
        NormalSingleton s1 = NormalSingleton.getInstance();

        Constructor<NormalSingleton> c = 
            NormalSingleton.class.getDeclaredConstructor();
        c.setAccessible(true);
        NormalSingleton s2 = c.newInstance();

        System.out.println(s1 == s2);  // false ❌ BROKEN!
    }
}
```

**Output:** `false` → Singleton is broken!

#### Example 2: Breaking with Serialization

```java
public class SerializationBreak {
    public static void main(String[] args) throws Exception {
        NormalSingleton s1 = NormalSingleton.getInstance();

        // Serialize
        ObjectOutputStream oos = 
            new ObjectOutputStream(new FileOutputStream("obj.txt"));
        oos.writeObject(s1);
        oos.close();

        // Deserialize
        ObjectInputStream ois = 
            new ObjectInputStream(new FileInputStream("obj.txt"));
        NormalSingleton s2 = (NormalSingleton) ois.readObject();

        System.out.println(s1 == s2);  // false ❌ BROKEN!
    }
}
```

**Output:** `false` → New instance created!

---

### ✅ Enum Singleton Cannot Be Broken

#### Try to break with Reflection:

```java
Constructor<?>[] constructors = 
    EnumSingleton.class.getDeclaredConstructors();

for (Constructor<?> c : constructors) {
    c.setAccessible(true);
    c.newInstance();  // BOOM 💥
}
```

**Output:**
```
Exception in thread "main"
java.lang.IllegalArgumentException:
Cannot reflectively create enum objects
```

✅ JVM blocks it!

#### Try to break with Serialization:

```java
EnumSingleton s1 = EnumSingleton.INSTANCE;

// Serialize and deserialize
ObjectOutputStream oos = ...
oos.writeObject(s1);

ObjectInputStream ois = ...
EnumSingleton s2 = (EnumSingleton) ois.readObject();

System.out.println(s1 == s2);  // true ✅
```

**Output:** `true` → Same instance reused!

---

### 📊 Singleton Comparison

| Approach | Thread Safe | Serialization Safe | Reflection Safe | Recommended |
|----------|-------------|-------------------|-----------------|-------------|
| Basic Lazy | ❌ | ❌ | ❌ | No |
| Synchronized | ✅ | ❌ | ❌ | Slow |
| Double-Checked Locking | ✅ | ❌ | ❌ | Good |
| **Enum Singleton** | ✅ | ✅ | ✅ | ⭐ **Best** |

---

### 🔥 Spring Boot Connection

```java
@Service
public class OrderService { }
```

**By default, Spring beans are Singletons!**

You use it every day! 😄

---

### 🧠 Memory Trick

**Singleton = one object, one responsibility, one truth**

---

### 🎯 Interview Answers

**Q: What is Singleton?**
> "Singleton is used when exactly one instance of a class is required to coordinate actions across the system."

**Q: What's the best way to implement Singleton?**
> "Enum-based Singleton is the most robust implementation because it is thread-safe, serialization-safe, and reflection-safe by JVM design."

---

## 6. Builder Pattern

### 🤔 What is Builder Pattern?

**Simple Definition:**
> Builder Pattern helps us create an object **step by step** in a clean and readable way.

---

### 🚨 The Problem Builder Solves

#### Problem 1: Telescoping Constructor

```java
class User {
    String name;
    int age;
    String email;
    String phone;
    String address;

    User(String name) { }
    User(String name, int age) { }
    User(String name, int age, String email) { }
    User(String name, int age, String email, String phone) { }
    User(String name, int age, String email, String phone, String address) { }
}
```

**Problems:**
- ❌ Too many constructors
- ❌ Confusing order
- ❌ Hard to read and maintain
- ❌ Optional fields nightmare

---

#### Problem 2: Constructor Confusion

```java
new User("Ali", 25, "a@gmail.com", null, "Address");
```

**Questions in your head:**
- What is `25`? (age? id? something else?)
- What is `null`? (email? phone?)
- What if I mix up the order?
- Order mistake = bug ❌

---

### 💡 Builder Pattern Solution

**Think of it like ordering food 🍔**

You choose only what you want, step by step.

**Clean, Readable Code:**
```java
User user = User.builder()
                .name("Ali")
                .age(25)
                .email("a@gmail.com")
                .build();
```

**Benefits:**
- ✓ Easy to read
- ✓ No confusion about order
- ✓ No wrong parameter placement
- ✓ Optional fields are truly optional

---

### 📝 Implementation Examples

#### Simple Version (For Learning)

```java
class User {
    String name;
    int age;
    String email;

    private User() {}  // Private constructor

    static class Builder {
        private User user = new User();

        Builder name(String name) {
            user.name = name;
            return this;  // Returns 'this' for chaining
        }

        Builder age(int age) {
            user.age = age;
            return this;
        }

        Builder email(String email) {
            user.email = email;
            return this;
        }

        User build() {
            return user;
        }
    }

    static Builder builder() {
        return new Builder();
    }
}
```

**Usage:**
```java
User user = User.builder()
                .name("Ali")
                .age(25)
                .build();

System.out.println(user.name);  // Ali
```

**⚠️ Issues with Simple Version:**
- Object is mutable (can be changed after creation)
- Builder stores the actual object (not individual fields)
- No validation before object creation
- Not thread-safe

---

#### Professional Version (Production-Ready) ⭐

```java
public class User {
    // Final fields = immutable
    private final String name;
    private final int age;
    private final String email;
    private final String phone;

    // Private constructor - accepts Builder
    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.phone = builder.phone;
    }

    // Static factory method
    public static Builder builder() {
        return new Builder();
    }

    // Static nested Builder class
    public static class Builder {
        // Builder stores individual FIELDS, not the object
        private String name;
        private int age;
        private String email;
        private String phone;

        private Builder() {}

        // Fluent API - return 'this' for chaining
        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        // Build method with validation
        public User build() {
            // Validation before creating object
            if (name == null || name.isEmpty()) {
                throw new IllegalStateException("Name is required");
            }
            if (age < 0) {
                throw new IllegalStateException("Age cannot be negative");
            }
            return new User(this);
        }
    }

    // Getters only (no setters - immutable)
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    public String getPhone() { return phone; }

    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + 
               ", email='" + email + "', phone='" + phone + "'}";
    }
}
```

**Usage:**
```java
User user = User.builder()
                .name("Ali")
                .age(30)
                .email("ali@gmail.com")
                .phone("1234567890")
                .build();

System.out.println(user);
// Output: User{name='Ali', age=30, email='ali@gmail.com', phone='1234567890'}
```

---

### 🎓 Real-World Example: Student Management

```java
public class Student {
    private final String name;
    private final int age;
    private final String email;
    private final String batch;
    private final String phone;

    private Student(StudentBuilder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.batch = builder.batch;
        this.phone = builder.phone;
    }

    public static StudentBuilder builder() {
        return new StudentBuilder();
    }

    public static class StudentBuilder {
        private String name;
        private int age;
        private String email;
        private String batch;
        private String phone;

        private StudentBuilder() {}

        public StudentBuilder name(String name) {
            this.name = name;
            return this;
        }

        public StudentBuilder age(int age) {
            this.age = age;
            return this;
        }

        public StudentBuilder email(String email) {
            this.email = email;
            return this;
        }

        public StudentBuilder batch(String batch) {
            this.batch = batch;
            return this;
        }

        public StudentBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Student build() {
            // Comprehensive validation
            if (name == null || name.trim().isEmpty()) {
                throw new IllegalStateException("Name is required");
            }
            if (age < 18 || age > 100) {
                throw new IllegalStateException("Age must be between 18 and 100");
            }
            if (email == null || !email.contains("@")) {
                throw new IllegalStateException("Valid email is required");
            }
            return new Student(this);
        }
    }

    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    public String getBatch() { return batch; }
    public String getPhone() { return phone; }

    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + 
               ", email='" + email + "', batch='" + batch + "'}";
    }
}
```

**Usage:**
```java
// With all fields
Student student1 = Student.builder()
                    .name("Ali Ahmed")
                    .age(22)
                    .email("ali@university.edu")
                    .phone("9876543210")
                    .batch("2024-CS-A")
                    .build();

// With only required fields
Student student2 = Student.builder()
                    .name("Sara Khan")
                    .age(21)
                    .email("sara@university.edu")
                    .build();
```

---

### ⚠️ Common Mistakes to Avoid

#### ❌ Mistake 1: Builder Stores Object Instead of Fields

**Bad:**
```java
public static class Builder {
    private User user = new User();  // ❌ Stores object
    
    public Builder name(String name) {
        user.name = name;  // Modifies object directly
        return this;
    }
}
```

**Good:**
```java
public static class Builder {
    private String name;  // ✅ Stores individual fields
    
    public Builder name(String name) {
        this.name = name;  // Modifies builder field
        return this;
    }
    
    public User build() {
        return new User(this);  // Creates object only when ready
    }
}
```

**Why it's better:**
- Object created only once (in `build()`)
- Better validation before object creation
- True immutability

---

#### ❌ Mistake 2: Mutable Object (No `final`)

**Bad:**
```java
public class User {
    private String name;  // ❌ Can be changed
    
    public void setName(String name) {
        this.name = name;
    }
}
```

**Good:**
```java
public class User {
    private final String name;  // ✅ Cannot be changed
    
    // No setters - only getters
    public String getName() {
        return name;
    }
}
```

---

#### ❌ Mistake 3: No Validation

**Bad:**
```java
public User build() {
    return new User(this);  // ❌ No validation
}
```

**Good:**
```java
public User build() {
    if (name == null || name.isEmpty()) {
        throw new IllegalStateException("Name is required");
    }
    if (age < 0) {
        throw new IllegalStateException("Age cannot be negative");
    }
    return new User(this);
}
```

---

#### ❌ Mistake 4: Using `set` Prefix

**Acceptable but not ideal:**
```java
public Builder setName(String name) {  // ❌ Too verbose
    this.name = name;
    return this;
}

// Usage - verbose
User.builder().setName("Ali").setAge(25).build();
```

**Better (Industry Standard):**
```java
public Builder name(String name) {  // ✅ Cleaner
    this.name = name;
    return this;
}

// Usage - cleaner!
User.builder().name("Ali").age(25).build();
```

---

### 🔒 Why Immutability Matters in Builder

#### What is Immutability?

**Immutable = Cannot be changed after creation**

```java
String s = "hello";
s = s.toUpperCase();  // Creates NEW object, doesn't change "hello"
```

---

#### The Problem Without Immutability

**Mutable class (BAD):**
```java
class User {
    String name;  // No 'final'
    int age;
}

// Builder creates it
User u = User.builder().name("Ali").age(25).build();

// Later someone can do:
u.age = -100;   // ❌ Breaks business rules!
```

**Builder's validation is now useless!**

---

#### Builder + Immutability = Perfect Combo ✅

**Two Phases:**

1. **Builder Phase** (Construction)
   - Object is under construction
   - Changes allowed
   - Validation happens

2. **After `build()`** (Usage)
   - Object is final
   - No changes allowed
   - Thread-safe

---

#### How to Make Immutable Object

**4 Rules:**

1. Use `private final` fields
2. No setters (only getters)
3. Private constructor (accepts Builder)
4. Values set only once via Builder

**Example:**
```java
class User {
    private final String name;  // Can't change
    private final int age;      // Can't change

    private User(Builder b) {
        this.name = b.name;
        this.age = b.age;
    }
    
    // Getters only, no setters
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

**After build:**
```java
User u = User.builder().name("Ali").age(25).build();
// u.age = 30;  ❌ Compile error - no setter!
```

---

#### Real-Life Analogy 🏗️

**Think of house construction:**

| Phase | Builder Pattern |
|-------|----------------|
| **Construction Phase** 🏗️ | Change rooms, walls freely |
| **After Completion** 🏠 | Cannot move walls randomly |

**Builder = construction process**  
**Immutable object = finished house**

---

#### Problems Without Immutability

| Problem | Impact |
|---------|--------|
| Anyone can change object later | ❌ Bugs are hard to trace |
| Not thread-safe | ❌ Concurrent modifications |
| Validation bypassed | ❌ Invalid states possible |
| Unexpected behavior | ❌ Hard to debug |

---

### 📊 Builder Implementation Comparison

| Feature | Simple Version | Professional Version |
|---------|---------------|---------------------|
| **Object Storage** | Stores object | Stores individual fields |
| **Immutability** | Mutable (no `final`) | Immutable (`final` fields) |
| **Thread Safety** | Not thread-safe | Thread-safe |
| **Validation** | No validation | Validation in `build()` |
| **Object Creation** | Early (in constructor) | Late (in `build()`) |
| **Method Names** | `setName()` | `name()` ✅ |
| **Constructor** | No-arg | Accepts Builder |
| **Best For** | Learning/prototyping | Production code ⭐ |

---

### ✅ Why Builder Works So Well

#### 1. Readability ✨

```java
.age(30).email("x@gmail.com")
```
Crystal clear what each value means!

---

#### 2. Optional Fields

```java
User user = User.builder()
                .name("Ali")
                .build();  // age and email skipped - no problem!
```

---

#### 3. Immutable Object

- Fields are `final`
- No setters
- Thread-safe

---

#### 4. Validation in One Place

```java
public User build() {
    // All validation logic here
    if (age < 0) throw new IllegalStateException("Invalid age");
    if (name == null) throw new IllegalStateException("Name required");
    return new User(this);
}
```

---

### 🔄 How Builder Pattern Works

**Step-by-Step Flow:**

```
Step 1: Get Builder
├─> User.builder()
│
Step 2: Set Properties (fluent chain)
├─> .name("Ali")
├─> .age(25)
├─> .email("ali@gmail.com")
│
Step 3: Validate & Build
├─> .build()
│   ├─> Validate all fields
│   ├─> Create new User(this)
│   └─> Return immutable User object
│
Step 4: Get Immutable Object
└─> User object (final fields, no setters)
```

**Visual Flow:**
```
┌─────────────────────────────┐
│  User.builder()             │
│  Creates Builder instance   │
└──────────┬──────────────────┘
           ↓
┌─────────────────────────────┐
│  .name("Ali")               │
│  Builder.name = "Ali"       │
│  return this (for chaining) │
└──────────┬──────────────────┘
           ↓
┌─────────────────────────────┐
│  .build()                   │
│  1. Validate fields         │
│  2. Create User(this)       │
│  3. Return immutable User   │
└──────────┬──────────────────┘
           ↓
┌─────────────────────────────┐
│  ✓ Immutable User Object    │
│  ✓ All fields final         │
│  ✓ No setters               │
│  ✓ Thread-safe              │
└─────────────────────────────┘
```

---

### 🌍 Real-World Examples

#### 1. StringBuilder (Classic Builder)

```java
String s = new StringBuilder()
            .append("Hello")
            .append(" ")
            .append("World")
            .toString();
```

Same idea → build step by step!

---

#### 2. Lombok Builder (Industry Standard)

```java
@Builder
public class Order {
    String id;
    double price;
    String customer;
}

// Usage - Lombok generates the Builder!
Order order = Order.builder()
                   .id("ORD-123")
                   .price(99.99)
                   .customer("Ali")
                   .build();
```

---

### 🎯 When to Use Builder Pattern?

#### ✅ Use Builder When:

- Object has **many optional fields** (3+)
- Object must be **immutable**
- Constructor becomes **ugly/complex**
- You want **readable, maintainable code**
- Need **validation** before object creation
- Building **complex objects** step-by-step

---

#### ❌ Don't Use When:

- Object has only **2-3 simple fields**
- **No optional fields**
- Object is simple

**Example when NOT to use:**
```java
// Simple class - just use constructor
class Point {
    final int x;
    final int y;
    
    Point(int x, int y) {  // ✅ Simple constructor is better
        this.x = x;
        this.y = y;
    }
}

Point p = new Point(10, 20);  // Clear and simple
```

---

### 📋 Builder Pattern Checklist

When implementing Builder, ensure:

- [ ] Private constructor in main class
- [ ] Static nested `Builder` class
- [ ] Builder stores **fields**, not object
- [ ] Fluent methods that `return this`
- [ ] `build()` method with validation
- [ ] Main class fields are `final`
- [ ] Main class has getters only (no setters)
- [ ] Static factory method `builder()`

---

### 🧠 Memory Tricks

| Constructor | Builder |
|------------|---------|
| Dump everything at once | Add things one by one, then build |

**Visual:** Think of **ordering food** 🍔

1. Choose bun
2. Choose patty
3. Choose toppings
4. Finally → **build()** your burger!

---

### 🎯 Interview Answers

**Q: What is Builder Pattern?**
> "Builder Pattern is used to construct complex objects step by step while keeping the object immutable and avoiding constructor overloads."

**Q: Why do we need Builder?**
> "Builder solves the telescoping constructor problem and provides a fluent API for creating objects with many optional fields while ensuring immutability and validation."

**Q: Why should the product class be immutable?**
> "Builder creates the object; immutability protects it after creation. This ensures thread-safety, prevents bugs from unexpected modifications, and guarantees the object remains in a valid state."

---

## 7. Prototype Pattern

### 🤔 What is Prototype Pattern?

**Simple Definition:**
> Prototype Pattern creates new objects by **cloning** an existing object instead of creating from scratch.

**Think of it like:** Making photocopies 📄 → Faster than writing everything again!

---

### 🚨 The Problem Prototype Solves

#### Problem 1: Expensive Object Creation

**Imagine creating this object:**
```java
DatabaseConnection db = new DatabaseConnection();
db.connect("server1.com", 3306);
db.authenticate("user", "password");
db.loadConfiguration();
db.initializePools(100);
// Takes 5 seconds to create! ⏰
```

**Now you need 10 more similar connections:**
```java
// Creating each from scratch = 5 seconds × 10 = 50 seconds! 😱
```

**Solution with Prototype:**
```java
DatabaseConnection db1 = original.clone();  // Just 0.1 second! ⚡
DatabaseConnection db2 = original.clone();
DatabaseConnection db3 = original.clone();
// Total: ~0.3 seconds instead of 50! 🚀
```

---

#### Problem 2: Complex Object Initialization

**Complex object with many steps:**
```java
GameCharacter hero = new GameCharacter();
hero.setName("Warrior");
hero.setHealth(100);
hero.setStrength(50);
hero.setDefense(30);
hero.loadTextures();        // Heavy operation
hero.loadAnimations();      // Heavy operation
hero.loadSounds();          // Heavy operation
hero.calculateStats();
hero.equipDefaultGear();
// 20+ lines of initialization code! 😰
```

**Need 100 similar warriors? Repeat this 100 times?** ❌

**Solution with Prototype:**
```java
GameCharacter warrior1 = templateWarrior.clone();  // Done! ✅
GameCharacter warrior2 = templateWarrior.clone();
GameCharacter warrior3 = templateWarrior.clone();
// Just modify what's different, e.g., name
```

---

### 💡 Prototype Pattern Solution

**Key Idea:**
1. Create a **template object** (prototype) once
2. **Clone** it whenever you need a copy
3. Modify the clone if needed

**Benefits:**
- ✓ Much faster than creating from scratch
- ✓ Hides complex creation logic
- ✓ No need to know all initialization steps
- ✓ Reduces memory if objects share data

---

### 🔧 How Prototype Works

#### The Cloneable Interface

In Java, use the `Cloneable` interface and override `clone()` method:

```java
public class MyClass implements Cloneable {
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

---

### 📝 Implementation Examples

#### Example 1: Basic Prototype (Shallow Copy)

```java
public class Employee implements Cloneable {
    private String name;
    private int age;
    private String department;

    public Employee(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }

    // Clone method
    @Override
    public Employee clone() {
        try {
            return (Employee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Clone not supported", e);
        }
    }

    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public String getDepartment() { return department; }

    @Override
    public String toString() {
        return "Employee{name='" + name + "', age=" + age + 
               ", department='" + department + "'}";
    }
}
```

**Usage:**
```java
// Create original
Employee original = new Employee("Ali", 30, "IT");

// Clone it
Employee clone1 = original.clone();
Employee clone2 = original.clone();

// Modify clones
clone1.setName("Sara");
clone2.setName("Ahmed");

System.out.println(original);  // Employee{name='Ali', age=30, department='IT'}
System.out.println(clone1);    // Employee{name='Sara', age=30, department='IT'}
System.out.println(clone2);    // Employee{name='Ahmed', age=30, department='IT'}

// They are different objects
System.out.println(original == clone1);  // false ✅
```

---

#### Example 2: Game Character (Real-World)

```java
public class GameCharacter implements Cloneable {
    private String type;        // "Warrior", "Mage", "Archer"
    private int health;
    private int strength;
    private int defense;
    private String texture;     // Image file path

    public GameCharacter(String type, int health, int strength, 
                        int defense, String texture) {
        this.type = type;
        this.health = health;
        this.strength = strength;
        this.defense = defense;
        this.texture = texture;
        
        // Simulate expensive operations
        System.out.println("Loading textures for " + type + "... (expensive!)");
        System.out.println("Loading animations for " + type + "... (expensive!)");
    }

    @Override
    public GameCharacter clone() {
        try {
            System.out.println("Cloning " + type + "... (fast!)");
            return (GameCharacter) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public String getType() { return type; }
    public int getHealth() { return health; }
    public void setHealth(int health) { this.health = health; }

    @Override
    public String toString() {
        return type + " {health=" + health + ", strength=" + strength + 
               ", defense=" + defense + "}";
    }
}
```

**Usage:**
```java
// Create template warrior (expensive - done once)
System.out.println("=== Creating Template ===");
GameCharacter warriorTemplate = new GameCharacter("Warrior", 100, 50, 30, "warrior.png");

System.out.println("\n=== Creating Army via Cloning ===");
// Create army of warriors (fast - cloning)
GameCharacter warrior1 = warriorTemplate.clone();
GameCharacter warrior2 = warriorTemplate.clone();
GameCharacter warrior3 = warriorTemplate.clone();

// Customize if needed
warrior1.setHealth(90);
warrior2.setHealth(95);

System.out.println(warrior1);
System.out.println(warrior2);
System.out.println(warrior3);
```

**Output:**
```
=== Creating Template ===
Loading textures for Warrior... (expensive!)
Loading animations for Warrior... (expensive!)

=== Creating Army via Cloning ===
Cloning Warrior... (fast!)
Cloning Warrior... (fast!)
Cloning Warrior... (fast!)
Warrior {health=90, strength=50, defense=30}
Warrior {health=95, strength=50, defense=30}
Warrior {health=100, strength=50, defense=30}
```

---

### 🔍 Shallow Copy vs Deep Copy

#### Shallow Copy (Default `clone()`)

**Problem:**
- Copies primitive types (`int`, `String`, etc.)
- **Shares references** for objects

**Example:**
```java
public class Person implements Cloneable {
    private String name;
    private Address address;  // Object reference
    
    @Override
    public Person clone() {
        return (Person) super.clone();  // Shallow copy
    }
}

Person p1 = new Person("Ali", new Address("Street 1"));
Person p2 = p1.clone();

p2.address.setStreet("Street 2");

System.out.println(p1.address);  // Street 2 ❌ (Changed!)
System.out.println(p2.address);  // Street 2
```

**Both share the same `Address` object!**

---

#### Deep Copy (Manual Implementation)

**Solution:** Clone nested objects too

```java
public class Person implements Cloneable {
    private String name;
    private Address address;

    @Override
    public Person clone() {
        try {
            Person cloned = (Person) super.clone();
            // Clone the nested object too
            cloned.address = this.address.clone();  // Deep copy
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

public class Address implements Cloneable {
    private String street;

    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
    
    // Getters and setters
}
```

**Now:**
```java
Person p1 = new Person("Ali", new Address("Street 1"));
Person p2 = p1.clone();

p2.address.setStreet("Street 2");

System.out.println(p1.address);  // Street 1 ✅ (Unchanged!)
System.out.println(p2.address);  // Street 2 ✅
```

---

### 📊 Shallow vs Deep Copy Comparison

| Feature | Shallow Copy | Deep Copy |
|---------|-------------|-----------|
| **Primitive types** | ✅ Copies values | ✅ Copies values |
| **String** | ✅ Copies reference (immutable, so safe) | ✅ Copies reference |
| **Object references** | ❌ Shares same object | ✅ Creates new object |
| **Implementation** | `super.clone()` | Manual cloning of nested objects |
| **Use when** | No nested objects | Has nested mutable objects |

---

### 🎓 Real-World Example: Document Templates

```java
public class Document implements Cloneable {
    private String title;
    private String content;
    private String header;
    private String footer;
    private List<String> styles;

    public Document(String title) {
        this.title = title;
        // Expensive operations
        this.header = loadHeader();
        this.footer = loadFooter();
        this.styles = loadDefaultStyles();
        System.out.println("Creating new document template (expensive)...");
    }

    private String loadHeader() {
        // Simulate expensive DB/file operation
        return "=== Company Header ===";
    }

    private String loadFooter() {
        return "=== Company Footer ===";
    }

    private List<String> loadDefaultStyles() {
        return new ArrayList<>(Arrays.asList("Bold", "Italic", "Underline"));
    }

    @Override
    public Document clone() {
        try {
            System.out.println("Cloning document (fast)...");
            Document cloned = (Document) super.clone();
            // Deep copy for list
            cloned.styles = new ArrayList<>(this.styles);
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public void setTitle(String title) { this.title = title; }
    public void setContent(String content) { this.content = content; }

    @Override
    public String toString() {
        return "Document{title='" + title + "', content='" + content + "'}";
    }
}
```

**Usage:**
```java
// Create template once (expensive)
Document template = new Document("Company Template");

// Create many documents by cloning (fast)
Document doc1 = template.clone();
doc1.setTitle("Invoice #001");
doc1.setContent("Invoice details...");

Document doc2 = template.clone();
doc2.setTitle("Report Q1");
doc2.setContent("Quarterly report...");

Document doc3 = template.clone();
doc3.setTitle("Letter");
doc3.setContent("Dear customer...");

System.out.println(doc1);
System.out.println(doc2);
System.out.println(doc3);
```

---

### 🎮 Another Example: Game Enemies

```java
public class Enemy implements Cloneable {
    private String type;
    private int health;
    private int damage;
    private String aiScript;     // Complex AI behavior
    private Sprite sprite;       // Heavy graphics object

    public Enemy(String type, int health, int damage) {
        this.type = type;
        this.health = health;
        this.damage = damage;
        
        // Expensive operations
        this.aiScript = loadAIScript(type);
        this.sprite = loadSprite(type);
        System.out.println("Creating " + type + " (expensive operation)...");
    }

    private String loadAIScript(String type) {
        // Simulate loading from file
        return type + "_ai_script.lua";
    }

    private Sprite loadSprite(String type) {
        // Simulate loading heavy texture
        return new Sprite(type + ".png");
    }

    @Override
    public Enemy clone() {
        try {
            System.out.println("Cloning " + type + " (instant)...");
            return (Enemy) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public void setHealth(int health) { this.health = health; }

    @Override
    public String toString() {
        return type + " {health=" + health + ", damage=" + damage + "}";
    }
}

class Sprite {
    String filename;
    Sprite(String filename) { this.filename = filename; }
}
```

**Usage:**
```java
// Create enemy templates (expensive - done once at game start)
Enemy zombieTemplate = new Enemy("Zombie", 50, 10);
Enemy orcTemplate = new Enemy("Orc", 100, 20);

// Spawn enemies during gameplay (fast - cloning)
Enemy zombie1 = zombieTemplate.clone();
Enemy zombie2 = zombieTemplate.clone();
Enemy zombie3 = zombieTemplate.clone();

Enemy orc1 = orcTemplate.clone();
Enemy orc2 = orcTemplate.clone();

// Each can have different health after damage
zombie1.setHealth(30);
orc1.setHealth(75);
```

---

### 🎯 When to Use Prototype Pattern?

#### ✅ Use Prototype When:

- Object creation is **expensive** (time/resources)
- Objects are **similar** with minor differences
- Need to **avoid complex initialization** code
- Creating many **instances of similar objects**
- **Performance** is critical
- Want to hide **complex creation logic**

**Common Use Cases:**
- Game characters/enemies/items
- Document templates
- Database connection pools
- UI component templates
- Configuration objects
- Cache entries

---

#### ❌ Don't Use When:

- Object creation is **cheap/simple**
- Objects are **very different** from each other
- **No repeated patterns** in object creation
- Deep copying is **too complex** to implement

**Example when NOT to use:**
```java
// Simple object - just use constructor
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }
}

Point p1 = new Point(10, 20);  // ✅ Simple constructor is better
Point p2 = new Point(30, 40);
```

---

### ⚠️ Important Considerations

#### 1. Cloneable Interface

```java
public class MyClass implements Cloneable {  // Must implement
    @Override
    public MyClass clone() {
        try {
            return (MyClass) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);  // Handle exception
        }
    }
}
```

---

#### 2. Deep Copy for Complex Objects

**Always deep copy mutable nested objects:**
```java
@Override
public Person clone() {
    Person cloned = (Person) super.clone();
    cloned.address = this.address.clone();        // Deep copy
    cloned.skills = new ArrayList<>(this.skills); // Deep copy
    return cloned;
}
```

---

#### 3. Prototype Registry (Advanced)

**For managing multiple prototypes:**
```java
public class PrototypeRegistry {
    private Map<String, GameCharacter> prototypes = new HashMap<>();

    public void registerPrototype(String key, GameCharacter prototype) {
        prototypes.put(key, prototype);
    }

    public GameCharacter getPrototype(String key) {
        GameCharacter prototype = prototypes.get(key);
        return prototype != null ? prototype.clone() : null;
    }
}
```

**Usage:**
```java
PrototypeRegistry registry = new PrototypeRegistry();

// Register templates
registry.registerPrototype("warrior", new GameCharacter("Warrior", 100, 50, 30, "w.png"));
registry.registerPrototype("mage", new GameCharacter("Mage", 70, 80, 20, "m.png"));

// Create instances
GameCharacter w1 = registry.getPrototype("warrior");
GameCharacter w2 = registry.getPrototype("warrior");
GameCharacter m1 = registry.getPrototype("mage");
```

---

### 📊 Prototype vs Other Patterns

| Feature | Prototype | Factory | Builder |
|---------|-----------|---------|---------|
| **Purpose** | Clone existing object | Create new object | Build complex object step-by-step |
| **Creation** | Copy from prototype | Factory method | Builder methods |
| **Performance** | ⚡ Fast (cloning) | Normal | Normal |
| **Complexity** | Low | Medium | High |
| **Use Case** | Many similar objects | Different types | Complex construction |
| **When** | Objects are expensive to create | Need polymorphism | Many optional fields |

---

### 🔄 How Prototype Pattern Works

**Step-by-Step Flow:**

```
Step 1: Create Template Object (Prototype)
├─> GameCharacter template = new GameCharacter(...)
│   ├─> Load textures (expensive)
│   ├─> Load animations (expensive)
│   └─> Initialize complex state
│
Step 2: Clone When Needed
├─> GameCharacter clone1 = template.clone()
│   └─> Memory copy (fast!) ⚡
│
Step 3: Customize Clone
├─> clone1.setHealth(90)
└─> clone1.setName("Hero1")
```

**Visual Flow:**
```
┌─────────────────────────────────┐
│  Template Object (Created Once) │
│  ┌─────────────────────────┐    │
│  │ GameCharacter           │    │
│  │ - type: "Warrior"       │    │
│  │ - health: 100           │    │
│  │ - textures: [loaded]    │    │
│  │ - animations: [loaded]  │    │
│  └─────────────────────────┘    │
└────────────┬────────────────────┘
             │
             │ clone()  ⚡ (fast)
             ↓
┌─────────────────────────────────┐
│     Cloned Objects              │
├─────────────────────────────────┤
│  Clone 1  │  Clone 2  │ Clone 3 │
│  warrior1 │  warrior2 │ warrior3│
│  health:90│ health:95 │health:100│
└─────────────────────────────────┘
```

---

### 🧠 Memory Tricks

**Prototype = Xerox Machine** 📠
- Create master copy once
- Make photocopies when needed
- Each copy can be modified

**Think:**
> "Why write the same letter 100 times? Write once, photocopy 100 times!"

---

### 🎯 Interview Answers

**Q: What is Prototype Pattern?**
> "Prototype Pattern creates new objects by cloning an existing object rather than creating from scratch. It's useful when object creation is expensive or complex."

**Q: When would you use Prototype Pattern?**
> "Use Prototype when object creation is expensive, you need many similar objects, or you want to hide complex initialization. Common examples include game characters, document templates, and database connections."

**Q: What's the difference between Shallow and Deep Copy?**
> "Shallow copy copies primitive values but shares references to nested objects. Deep copy creates new instances of nested objects too. Use deep copy when nested objects are mutable to avoid unintended sharing."

**Q: How is Prototype different from Factory?**
> "Factory creates new objects from scratch and decides which class to instantiate, while Prototype clones existing objects for better performance. Factory focuses on 'what to create', Prototype focuses on 'how fast to create'."

---

## 8. Factory Pattern (Brief)

### 🏭 Factory Pattern

#### Key Idea
**Factory decides which object to create**

- Client does NOT use `new`
- Client depends on interface, not concrete class

#### Examples
- Payment methods (UPI / Card / Wallet)
- Notifications (Email / SMS / Push)
- Database connections (MySQL / PostgreSQL / MongoDB)

#### What it Solves
- **Where** to create an object
- **How** to create an object
- Hides creation complexity

---

## 9. Static vs Non-Static

### 🔧 Important Java Concept

Understanding static vs non-static is **crucial** for Singleton pattern.

---

### ❌ Rule 1: Static CANNOT Access Non-Static Directly

```java
class A {
    int x = 10;          // non-static

    static void test() {
        System.out.println(x); // ❌ Compile-time error!
    }
}
```

**Error:**
```
non-static variable x cannot be referenced from a static context
```

**Why?**
- `static` belongs to **class**
- `non-static` belongs to **object**
- Static code doesn't know **which object** to use

---

### ✅ Rule 2: Static CAN Access Non-Static Via Object

```java
class A {
    int x = 10;

    static void test() {
        A obj = new A();       // Create object
        System.out.println(obj.x); // ✅ Access via object
    }
}
```

**This is 100% valid!**

**You must:**
1. Create an object
2. Access via that object

---

### 🔍 Why Singleton Works

```java
class Singleton {
    private static Singleton instance;  // STATIC variable

    public static Singleton getInstance() {  // STATIC method
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;  // ✅ Static accessing static
    }
}
```

**Key Point:**
- Both `instance` and `getInstance()` are **static**
- Static can access static → perfectly valid ✅

---

### 📊 Access Rules Table

|  | **Static** | **Non-Static** |
|---|------------|----------------|
| **Static** | ✅ Direct access | ❌ Need object reference |
| **Non-Static** | ✅ Direct access | ✅ Direct access |

---

### 🧠 Memory Rule

> **Static talks only to static — unless you bring an object**

---

### 🎯 Interview Answer

> "Static methods cannot access non-static members directly because static belongs to the class while non-static belongs to objects. In Singleton, the instance variable is static, so the static `getInstance()` method can safely access and return it."

---

## 10. Comparison Tables

### 📊 Creational Patterns at a Glance

| Pattern | When | How | Where | How Many | Immutability |
|---------|------|-----|-------|----------|--------------|
| **Singleton** | Once | `getInstance()` | Static method | **One** | Usually |
| **Builder** | Step-by-step | `.name().age().build()` | Builder class | One | ✅ Yes |
| **Factory** | Runtime | `factory.create()` | Factory class | Many | Depends |
| **Prototype** | Clone | `clone()` | Existing object | Many | Depends |

---

### 📊 Pattern Purpose Summary

| Pattern | Main Question | Key Benefit |
|---------|--------------|-------------|
| **Singleton** | How many objects? → only one | Resource management, consistency |
| **Builder** | How to create? → step by step | Clean, readable, immutable code |
| **Factory** | Where & how to create? → factory decides | Loose coupling, flexibility |
| **Prototype** | How to create copies? → clone existing | Performance, efficiency |

---

### 📊 Singleton Implementations

| Type | Thread Safe | Serialization Safe | Reflection Safe | Interview Value |
|------|-------------|-------------------|-----------------|-----------------|
| Basic Lazy | ❌ | ❌ | ❌ | Beginner |
| Synchronized | ✅ | ❌ | ❌ | Good |
| Double-Checked Locking | ✅ | ❌ | ❌ | Very Good |
| **Enum** | ✅ | ✅ | ✅ | ⭐ **Best Practice** |

---

### 📊 Builder vs Setter Pattern

| Feature | Builder | Setter |
|---------|---------|--------|
| **Immutability** | ✅ Immutable | ❌ Mutable |
| **Thread Safety** | ✅ Thread-safe | ❌ Not thread-safe |
| **Object State** | ✅ Always valid | ❌ Can be partially built |
| **API** | ✅ Clean, fluent | ❌ Error-prone |
| **Readability** | ✅ High | ❌ Low |
| **Validation** | ✅ Centralized | ❌ Scattered |

---

### 📊 Prototype vs Other Patterns

| Feature | Prototype | Factory | Builder |
|---------|-----------|---------|---------|
| **Purpose** | Clone existing object | Create new object | Build complex object |
| **Creation** | Copy from prototype | Factory method | Step-by-step |
| **Performance** | ⚡ Fast (cloning) | Normal | Normal |
| **Complexity** | Low | Medium | High |
| **Use Case** | Many similar objects | Different types | Complex construction |
| **Best For** | Expensive objects | Polymorphism | Many optional fields |

---

### 📊 Shallow vs Deep Copy

| Feature | Shallow Copy | Deep Copy |
|---------|-------------|-----------|
| **Primitive types** | ✅ Copies values | ✅ Copies values |
| **String** | ✅ Copies reference | ✅ Copies reference |
| **Object references** | ❌ Shares same object | ✅ Creates new object |
| **Implementation** | `super.clone()` | Manual cloning |
| **Performance** | ⚡ Faster | Slower |
| **Use when** | No nested objects | Has nested mutable objects |

---

## 11. Interview Guide

### 🎯 One-Line Definitions

#### SOLID & Design Patterns
> "SOLID principles are rules for good design; Design patterns are practical examples of those rules."

---

#### Three Types of Patterns
> "Creational patterns deal with object creation, Structural patterns deal with class structure, and Behavioral patterns deal with object interaction."

---

#### Creational Patterns
> "Creational design patterns answer when, how, where, and how many objects should be created, keeping client code simple, flexible, and loosely coupled."

---

#### Singleton Pattern
> "Singleton ensures exactly one instance of a class exists throughout the application to coordinate actions across the system."

---

#### Enum Singleton
> "Enum-based Singleton is the most robust implementation because it is thread-safe, serialization-safe, and reflection-safe by JVM design."

---

#### Builder Pattern
> "Builder Pattern constructs complex objects step by step while keeping the object immutable and avoiding constructor overloads. The Builder stores individual fields and validates them before creating the final immutable object."

---

#### Prototype Pattern
> "Prototype Pattern creates new objects by cloning an existing object rather than creating from scratch, which is useful when object creation is expensive or complex."

---

#### Static vs Non-Static
> "Static methods cannot access non-static members directly. In Singleton, the instance variable is static, so the static getInstance() method can safely access and return it."

---

### 🧠 Memory Tricks

| Pattern | Memory Trick |
|---------|-------------|
| **Singleton** | One object, one responsibility, one truth |
| **Builder** | Constructor = dump everything at once<br>Builder = add one by one, then build |
| **Prototype** | Xerox machine 📠 - create master, photocopy when needed |
| **Static** | Static talks only to static — unless you bring an object |

---

### 💡 Key Principles

#### Client Responsibility
> "Client should focus on business logic, not object creation"

Simply:
> "Use objects, don't create them in client code"

---

#### Four Core Questions
1. **When** should an object be created?
2. **How** should we create an object?
3. **Where** should we create an object?
4. **How many** objects should be created?

---

### 🔥 Spring Boot Connections

#### Singleton in Spring
```java
@Service
public class UserService {}
```
👉 Spring beans are Singleton by default

---

#### Dependency Injection
```java
@Autowired
PaymentService paymentService;
```
👉 Spring decides when, how, and where to create objects

---

#### Builder with Lombok
```java
@Builder
public class User {
    String name;
    int age;
}
```
👉 Lombok generates Builder code automatically

---

#### Prototype Scope in Spring
```java
@Component
@Scope("prototype")
public class ShoppingCart {}
```
👉 Spring creates new instance each time (like Prototype pattern)

---

### 📝 Common Interview Questions

#### Q1: What are Creational Design Patterns?

**Answer:**
> "Creational design patterns focus on when and how objects should be created, instead of directly using `new`, to make the system flexible and loosely coupled. The main patterns are Singleton, Factory, Builder, and Prototype."

---

#### Q2: How do you implement Singleton in Java?

**Answer:**
> "A Singleton is created by making the constructor private, creating a static instance variable, and providing a public static method to return the same instance every time. The best approach is using an enum, which is thread-safe, serialization-safe, and reflection-safe by JVM design."

---

#### Q3: Why use Builder Pattern?

**Short Answer:**
> "Builder Pattern is used to construct complex objects step by step while keeping the object immutable and avoiding constructor overloads. It's especially useful when an object has many optional fields."

**Extended Answer:**
> "Builder Pattern solves the telescoping constructor problem where you have multiple constructors with different parameter combinations. Instead, Builder provides a fluent API where you can set only the fields you need, validate them in one place (the `build()` method), and create an immutable object with `final` fields. The Builder stores individual fields rather than the object itself, which allows for better validation and true immutability."

---

#### Q4: What's the difference between Simple Builder and Professional Builder?

**Answer:**
> "Simple Builder stores the actual object and modifies it directly, making the object mutable. Professional Builder stores individual fields and creates the object only in the `build()` method, ensuring immutability by using `final` fields and validating before object creation."

**Key Differences:**

| Aspect | Simple | Professional |
|--------|--------|--------------|
| Storage | Stores object | Stores fields |
| Immutability | No | Yes (`final`) |
| Validation | No | In `build()` |
| Thread-safe | No | Yes |

---

#### Q5: Why does Builder return 'this'?

**Answer:**
> "Builder methods return `this` to enable method chaining (fluent API). This allows you to chain multiple method calls in a single statement, making the code more readable and expressive."

**Example:**
```java
User user = User.builder()
                .name("Ali")      // returns this
                .age(25)          // returns this
                .email("a@b.com") // returns this
                .build();         // returns User
```

---

#### Q6: What is Prototype Pattern and when would you use it?

**Answer:**
> "Prototype Pattern creates new objects by cloning an existing object rather than creating from scratch. It's used when object creation is expensive (time or resources), when you need many similar objects, or when you want to hide complex initialization logic. Common examples include game characters, document templates, and database connection pools."

---

#### Q7: What's the difference between Shallow Copy and Deep Copy?

**Answer:**
> "Shallow copy copies primitive values but shares references to nested objects, meaning changes to nested objects in the clone affect the original. Deep copy creates new instances of nested objects too, ensuring complete independence. Use shallow copy when objects have no nested mutable objects; use deep copy when they do."

**Example:**
```java
// Shallow copy - shares Address reference
Person clone = (Person) super.clone();

// Deep copy - creates new Address
Person clone = (Person) super.clone();
clone.address = this.address.clone();
```

---

#### Q8: How is Prototype different from Factory Pattern?

**Answer:**
> "Factory creates new objects from scratch and decides which class to instantiate based on input, while Prototype clones existing objects for better performance. Factory focuses on 'what type to create' (polymorphism), while Prototype focuses on 'how fast to create' (performance). Use Factory when you need different types of objects; use Prototype when you need many similar objects quickly."

---

#### Q9: Can static methods access non-static members?

**Answer:**
> "Static methods cannot access non-static members directly because static belongs to the class while non-static belongs to objects. However, they can access non-static members through an object reference. In Singleton, the instance variable is static, so the static getInstance() method can safely access it."

---

#### Q10: Why should the Builder product class be immutable?

**Answer:**
> "Builder creates the object; immutability protects it after creation. This ensures:
> - Thread-safety
> - No bugs from unexpected modifications
> - Object remains in a valid state
> - Validation rules cannot be bypassed"

---

### 🎓 Use Case Table

#### Singleton Use Cases

| Use Case | Why Singleton? |
|----------|---------------|
| Database connection | Expensive to create, should be shared |
| Logger | One log stream for entire application |
| Configuration | Single source of truth |
| Cache | Shared memory across application |
| Thread pool | Manage limited resources |

---

#### Builder Use Cases

**✅ Use Builder When:**
- Object has many optional fields (3+)
- Object must be immutable
- Constructor becomes ugly/complex
- You want readable, maintainable code
- Need validation before object creation

**❌ Don't Use When:**
- Object has only 2-3 simple fields
- No optional fields
- Object is simple

---

#### Prototype Use Cases

**✅ Use Prototype When:**
- Object creation is expensive (time/resources)
- Objects are similar with minor differences
- Need to avoid complex initialization
- Creating many instances of similar objects
- Performance is critical

**❌ Don't Use When:**
- Object creation is cheap/simple
- Objects are very different
- No repeated patterns
- Deep copying is too complex

---

## 📚 Summary

### 🎯 Main Takeaways

1. **SOLID principles** = Rules for good design
2. **Design patterns** = Practical solutions following those rules
3. **Creational patterns** = Control object creation

---

### 🔑 Four Core Questions

1. **When** → Singleton, Lazy Init
2. **How** → Builder, Factory, Prototype
3. **Where** → Factory, Dependency Injection
4. **How Many** → Singleton, Prototype

---

### ⭐ Best Practices

- **Use Enum Singleton** for thread-safe, reflection-safe implementation
- **Use Builder** for objects with many optional fields
- **Use Prototype** when object creation is expensive
- **Avoid `new`** in client code when possible
- **Let frameworks** (like Spring) manage object creation
- **Make objects immutable** when using Builder
- **Validate in one place** (Builder's `build()` method)
- **Use deep copy** when cloning objects with nested mutable objects

---

### 🧠 Golden Rules

> **Client should focus on business logic, not object creation**

> **Use objects, don't create them in client code**

> **Builder creates; Immutability protects**

> **Prototype = create once, clone many**

---

## 🚀 Next Steps

### To Master Creational Design Patterns:

1. ✅ Practice implementing all Singleton types
2. ✅ Practice implementing Builder pattern
3. ✅ Practice implementing Prototype with shallow and deep copy
4. ✅ Study Factory pattern in depth
5. ✅ Learn Abstract Factory
6. ✅ Map each pattern to SOLID principles
7. ✅ Practice with real-world examples
8. ✅ Study Spring Boot's use of these patterns
9. ✅ Convert existing classes to use patterns
10. ✅ Understand when NOT to use patterns

---

## 📖 Final Words

**Remember:** 

Design patterns are **tools**, not **rules**. 

Use them when they solve real problems in your code!

Don't force patterns where they don't fit.

---
