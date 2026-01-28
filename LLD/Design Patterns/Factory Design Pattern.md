# Factory Design Pattern - Complete Guide

## Table of Contents
1. [Introduction & Definition](#introduction--definition)
2. [Core Concept & Fundamentals](#core-concept--fundamentals)
3. [Problems Factory Pattern Solves](#problems-factory-pattern-solves)
4. [Types of Factory Patterns](#types-of-factory-patterns)
5. [Open/Closed Principle & Factory Pattern](#openclosed-principle--factory-pattern)
6. [Complete Working Examples](#complete-working-examples)
7. [When to Use Factory Pattern](#when-to-use-factory-pattern)
8. [Advantages & Disadvantages](#advantages--disadvantages)
9. [Best Practices](#best-practices)
10. [Real-World Use Cases](#real-world-use-cases)
11. [Quick Reference Guide](#quick-reference-guide)

---

## Introduction & Definition

### What is Factory Pattern?

**Factory Pattern** is a **creational design pattern** that provides an interface for creating objects in a superclass, but allows subclasses or factory classes to alter the type of objects that will be created.

### Formal Definition

> The Factory Design Pattern provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created. Instead of calling constructors directly using the `new` operator, you delegate object creation to a factory method or factory class.

### Key Components

1. **Creational Pattern**: Deals with object creation mechanisms
2. **Interface/Abstraction**: Provides a common way to create objects
3. **Delegation**: Object creation is delegated to factory methods or classes
4. **Flexibility**: The actual class being instantiated can be determined at runtime

### Simplified Definition

> Instead of creating objects directly using the `new` keyword, you ask a **factory** to create objects for you.

### Basic Concept

```java
// ❌ Traditional Way (Without Factory)
Pizza pizza = new CheesePizza();  // Direct instantiation

// ✅ Factory Way
Pizza pizza = PizzaFactory.createPizza("cheese");  // Delegated creation
```

**Key Idea:** Delegate object creation to a factory class/method rather than creating objects directly.

---

## Core Concept & Fundamentals

### Traditional Approach vs Factory Approach

#### Traditional Way (Without Factory)

```java
// Client directly creates objects
Pizza pizza1 = new CheesePizza();
Pizza pizza2 = new PepperoniPizza();
Pizza pizza3 = new VeggiePizza();
```

**Problems:**
- ❌ Client must know all concrete classes
- ❌ Hard to change which class gets instantiated
- ❌ Violates the principle of "programming to interfaces"
- ❌ Tight coupling between client and concrete classes

#### Factory Way

```java
// Client asks factory to create objects
Pizza pizza1 = PizzaFactory.createPizza("cheese");
Pizza pizza2 = PizzaFactory.createPizza("pepperoni");
Pizza pizza3 = PizzaFactory.createPizza("veggie");
```

**Benefits:**
- ✅ Client only knows about the factory and interface
- ✅ Easy to change or add new types
- ✅ Follows "programming to interfaces" principle
- ✅ Loose coupling between client and concrete classes

---

## Problems Factory Pattern Solves

### Problem 1: Tight Coupling Between Client and Concrete Classes

#### ❌ Without Factory Pattern

```java
public class PizzaStore {
    public void orderPizza(String type) {
        Pizza pizza;
        
        // Client is tightly coupled to concrete classes
        if (type.equals("cheese")) {
            pizza = new CheesePizza();      // Direct dependency
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();   // Direct dependency
        } else {
            pizza = new VeggiePizza();      // Direct dependency
        }
        
        pizza.prepare();
        pizza.bake();
    }
}
```

**Why This Is Bad:**
- PizzaStore class directly depends on CheesePizza, PepperoniPizza, VeggiePizza
- If pizza creation changes, must modify PizzaStore
- Cannot easily swap or add new pizza types
- Hard to test (cannot mock concrete classes easily)

#### ✅ With Factory Pattern

```java
public class PizzaStore {
    private PizzaFactory factory;
    
    public PizzaStore(PizzaFactory factory) {
        this.factory = factory;
    }
    
    public void orderPizza(String type) {
        // Loosely coupled - only knows about Pizza interface
        Pizza pizza = factory.createPizza(type);
        
        pizza.prepare();
        pizza.bake();
    }
}
```

**Why This Is Better:**
- ✅ PizzaStore only depends on Pizza interface and PizzaFactory
- ✅ Changes to pizza creation don't affect PizzaStore
- ✅ Easy to swap factory implementations
- ✅ Easy to test with mock factories

---

### Problem 2: Violation of Single Responsibility Principle

#### ❌ Without Factory Pattern

```java
public class OrderService {
    public void processOrder(Order order) {
        // Responsibility 1: Creating payment objects
        Payment payment;
        if (order.getPaymentMethod().equals("credit")) {
            payment = new CreditCardPayment();
            payment.setCardNumber(order.getCardNumber());
            payment.setExpiryDate(order.getExpiryDate());
            payment.setCVV(order.getCVV());
        } else if (order.getPaymentMethod().equals("paypal")) {
            payment = new PayPalPayment();
            payment.setEmail(order.getEmail());
            payment.setPassword(order.getPassword());
        }
        
        // Responsibility 2: Processing the order
        payment.process(order.getAmount());
        updateInventory();
        sendConfirmation();
    }
}
```

**Why This Is Bad:**
- ❌ OrderService has TWO responsibilities:
  1. Creating payment objects (with complex initialization)
  2. Processing orders
- ❌ Violates Single Responsibility Principle
- ❌ If payment creation changes, must modify OrderService
- ❌ Makes the class harder to understand and maintain

#### ✅ With Factory Pattern

```java
public class OrderService {
    private PaymentFactory paymentFactory;
    
    public OrderService(PaymentFactory paymentFactory) {
        this.paymentFactory = paymentFactory;
    }
    
    public void processOrder(Order order) {
        // Creation delegated to factory
        Payment payment = paymentFactory.createPayment(order);
        
        // Only responsible for order processing
        payment.process(order.getAmount());
        updateInventory();
        sendConfirmation();
    }
}
```

**Why This Is Better:**
- ✅ OrderService has ONE responsibility: processing orders
- ✅ Payment creation handled by PaymentFactory (single responsibility)
- ✅ Each class has a clear, focused purpose
- ✅ Easier to maintain and test

---

### Problem 3: Code Duplication Across Codebase

#### ❌ Without Factory Pattern

```java
// In OrderController.java
public void createOrder() {
    Database db;
    if (config.getDbType().equals("mysql")) {
        db = new MySQLDatabase();
        db.setHost("localhost");
        db.setPort(3306);
        db.setUsername("user");
        db.setPassword("pass");
    }
    // ... use database
}

// In ReportController.java
public void generateReport() {
    Database db;
    if (config.getDbType().equals("mysql")) {
        db = new MySQLDatabase();
        db.setHost("localhost");      // DUPLICATED CODE!
        db.setPort(3306);              // DUPLICATED CODE!
        db.setUsername("user");        // DUPLICATED CODE!
        db.setPassword("pass");        // DUPLICATED CODE!
    }
    // ... use database
}

// In UserController.java
public void getUsers() {
    Database db;
    if (config.getDbType().equals("mysql")) {
        db = new MySQLDatabase();
        db.setHost("localhost");      // DUPLICATED CODE!
        db.setPort(3306);              // DUPLICATED CODE!
        db.setUsername("user");        // DUPLICATED CODE!
        db.setPassword("pass");        // DUPLICATED CODE!
    }
    // ... use database
}
```

**Why This Is Bad:**
- ❌ Same creation code repeated everywhere
- ❌ If initialization changes, must update ALL locations
- ❌ Prone to errors (forgetting to update one location)
- ❌ Maintenance nightmare

#### ✅ With Factory Pattern

```java
// DatabaseFactory.java
public class DatabaseFactory {
    public static Database createDatabase() {
        String dbType = config.getDbType();
        
        if (dbType.equals("mysql")) {
            MySQLDatabase db = new MySQLDatabase();
            db.setHost("localhost");
            db.setPort(3306);
            db.setUsername("user");
            db.setPassword("pass");
            return db;
        }
        // ... handle other types
    }
}

// In OrderController.java
public void createOrder() {
    Database db = DatabaseFactory.createDatabase();  // ONE LINE!
    // ... use database
}

// In ReportController.java
public void generateReport() {
    Database db = DatabaseFactory.createDatabase();  // ONE LINE!
    // ... use database
}

// In UserController.java
public void getUsers() {
    Database db = DatabaseFactory.createDatabase();  // ONE LINE!
    // ... use database
}
```

**Why This Is Better:**
- ✅ Creation logic in ONE place
- ✅ Change once, affects everywhere
- ✅ No code duplication
- ✅ Consistent initialization across codebase

---

### Problem 4: Complex Object Creation Logic

#### ❌ Without Factory Pattern

```java
public void sendNotification(User user, String message) {
    // Complex logic to determine notification type
    Notification notification;
    
    if (user.isOnline() && user.hasAppInstalled()) {
        notification = new PushNotification();
        notification.setDeviceToken(user.getDeviceToken());
        notification.setSound("default");
        notification.setPriority("high");
    } else if (user.getEmail() != null && user.isEmailVerified()) {
        notification = new EmailNotification();
        notification.setSmtpServer("smtp.gmail.com");
        notification.setPort(587);
        notification.setFrom("noreply@example.com");
        notification.setTemplate("notification-template");
    } else if (user.getPhone() != null && user.getCountryCode() != null) {
        notification = new SMSNotification();
        notification.setGateway("twilio");
        notification.setApiKey(System.getenv("TWILIO_KEY"));
        notification.setFromNumber("+1234567890");
    } else {
        notification = new InboxNotification();
        notification.setExpiryDays(30);
    }
    
    notification.send(message);
}
```

**Why This Is Bad:**
- ❌ Complex creation logic mixed with business logic
- ❌ Hard to read and understand
- ❌ Difficult to test (how to test creation logic separately?)
- ❌ Business logic obscured by creation details

#### ✅ With Factory Pattern

```java
public void sendNotification(User user, String message) {
    // Simple, clean business logic
    Notification notification = NotificationFactory.createBestNotification(user);
    notification.send(message);
}

// Complex logic isolated in factory
class NotificationFactory {
    public static Notification createBestNotification(User user) {
        if (user.isOnline() && user.hasAppInstalled()) {
            return createPushNotification(user);
        } else if (user.getEmail() != null && user.isEmailVerified()) {
            return createEmailNotification();
        } else if (user.getPhone() != null && user.getCountryCode() != null) {
            return createSMSNotification();
        } else {
            return createInboxNotification();
        }
    }
    
    private static PushNotification createPushNotification(User user) {
        PushNotification notification = new PushNotification();
        notification.setDeviceToken(user.getDeviceToken());
        notification.setSound("default");
        notification.setPriority("high");
        return notification;
    }
    
    // Other creation methods...
}
```

**Why This Is Better:**
- ✅ Business logic is clear and simple
- ✅ Complex creation isolated in factory
- ✅ Each creation method can be tested independently
- ✅ Easy to modify creation logic without touching business code

---

### Problem 5: Difficult to Switch Implementations

#### ❌ Without Factory Pattern

```java
public class ReportGenerator {
    public void generateReport(Data data) {
        // Hardcoded to PDF
        PDFReport report = new PDFReport();
        report.setData(data);
        report.generate();
        report.save();
    }
}
```

**Problem Scenario:**
- Boss: "Now we need to support Excel reports too!"
- You: Must change ReportGenerator class
- Boss: "Sometimes PDF, sometimes Excel, based on user preference!"
- You: Must add if-else logic to ReportGenerator
- Boss: "Also add Word and HTML reports!"
- You: ReportGenerator becomes a mess!

#### ✅ With Factory Pattern

```java
public class ReportGenerator {
    public void generateReport(Data data, String format) {
        // Factory handles format selection
        Report report = ReportFactory.createReport(format);
        report.setData(data);
        report.generate();
        report.save();
    }
}

class ReportFactory {
    public static Report createReport(String format) {
        switch(format.toLowerCase()) {
            case "pdf": return new PDFReport();
            case "excel": return new ExcelReport();
            case "word": return new WordReport();
            case "html": return new HTMLReport();
            default: throw new IllegalArgumentException("Unknown format");
        }
    }
}
```

**Why This Is Better:**
- ✅ Easy to add new report formats
- ✅ ReportGenerator doesn't change when new formats added
- ✅ Format selection centralized in factory
- ✅ Can change format at runtime based on user preference

---

### Problem 6: Difficult to Test

#### ❌ Without Factory Pattern

```java
public class EmailService {
    public void sendEmail(String to, String message) {
        // Directly creates SMTP connection - can't test!
        SMTPConnection smtp = new SMTPConnection("smtp.gmail.com", 587);
        smtp.connect();
        smtp.authenticate("user@example.com", "password");
        smtp.send(to, message);
        smtp.disconnect();
    }
}

// How to test this?
@Test
public void testSendEmail() {
    EmailService service = new EmailService();
    service.sendEmail("test@example.com", "Test");  
    // This will actually try to connect to Gmail!
    // Cannot test without real SMTP server!
}
```

**Why This Is Bad:**
- ❌ Tests require actual SMTP connection
- ❌ Slow tests (network calls)
- ❌ Cannot test offline
- ❌ Cannot test error scenarios easily

#### ✅ With Factory Pattern (Using Dependency Injection)

```java
public class EmailService {
    private ConnectionFactory connectionFactory;
    
    public EmailService(ConnectionFactory factory) {
        this.connectionFactory = factory;
    }
    
    public void sendEmail(String to, String message) {
        SMTPConnection smtp = connectionFactory.createConnection();
        smtp.connect();
        smtp.authenticate();
        smtp.send(to, message);
        smtp.disconnect();
    }
}

// Easy to test with mock factory
@Test
public void testSendEmail() {
    // Create mock connection
    SMTPConnection mockConnection = mock(SMTPConnection.class);
    
    // Create mock factory
    ConnectionFactory mockFactory = mock(ConnectionFactory.class);
    when(mockFactory.createConnection()).thenReturn(mockConnection);
    
    // Test with mock
    EmailService service = new EmailService(mockFactory);
    service.sendEmail("test@example.com", "Test");
    
    // Verify behavior
    verify(mockConnection).connect();
    verify(mockConnection).send("test@example.com", "Test");
    // No actual network calls!
}
```

**Why This Is Better:**
- ✅ Tests run fast (no network calls)
- ✅ Can test offline
- ✅ Easy to test error scenarios with mocks
- ✅ Reliable and repeatable tests

---

### Problem 7: Conditional Object Creation Scattered Everywhere

#### ❌ Without Factory Pattern

```java
// In ShoppingCart.java
public void calculateShipping() {
    ShippingMethod shipping;
    if (totalWeight > 50) {
        shipping = new FreightShipping();
    } else if (isInternational) {
        shipping = new InternationalShipping();
    } else {
        shipping = new StandardShipping();
    }
}

// In OrderProcessor.java
public void processOrder() {
    ShippingMethod shipping;
    if (totalWeight > 50) {              // DUPLICATE LOGIC!
        shipping = new FreightShipping();
    } else if (isInternational) {         // DUPLICATE LOGIC!
        shipping = new InternationalShipping();
    } else {
        shipping = new StandardShipping();
    }
}

// In ShippingCalculator.java
public void estimateDelivery() {
    ShippingMethod shipping;
    if (totalWeight > 50) {              // DUPLICATE LOGIC!
        shipping = new FreightShipping();
    } else if (isInternational) {         // DUPLICATE LOGIC!
        shipping = new InternationalShipping();
    } else {
        shipping = new StandardShipping();
    }
}
```

**Why This Is Bad:**
- ❌ Same conditional logic repeated everywhere
- ❌ If shipping rules change (e.g., weight threshold changes to 40), must update ALL locations
- ❌ Easy to miss one location, causing bugs
- ❌ Inconsistent behavior if logic differs slightly in each place

#### ✅ With Factory Pattern

```java
class ShippingFactory {
    public static ShippingMethod createShipping(double weight, boolean isInternational) {
        if (weight > 50) {
            return new FreightShipping();
        } else if (isInternational) {
            return new InternationalShipping();
        } else {
            return new StandardShipping();
        }
    }
}

// In ShoppingCart.java
public void calculateShipping() {
    ShippingMethod shipping = ShippingFactory.createShipping(totalWeight, isInternational);
}

// In OrderProcessor.java
public void processOrder() {
    ShippingMethod shipping = ShippingFactory.createShipping(totalWeight, isInternational);
}

// In ShippingCalculator.java
public void estimateDelivery() {
    ShippingMethod shipping = ShippingFactory.createShipping(totalWeight, isInternational);
}
```

**Why This Is Better:**
- ✅ Conditional logic in ONE place
- ✅ Change once, affects everywhere automatically
- ✅ Consistent behavior across entire application
- ✅ Easy to modify shipping rules

---

### Summary: Problems Solved by Factory Pattern

| Problem | Without Factory | With Factory |
|---------|----------------|--------------|
| **Coupling** | Tight coupling to concrete classes | Loose coupling to interfaces |
| **Responsibility** | Classes have multiple responsibilities | Single responsibility per class |
| **Duplication** | Creation code duplicated everywhere | Creation logic centralized |
| **Complexity** | Complex creation mixed with business logic | Complex creation isolated |
| **Flexibility** | Hard to change implementations | Easy to swap implementations |
| **Testing** | Difficult to mock and test | Easy to test with mocks |
| **Maintenance** | Changes require modifying multiple files | Changes in one place |
| **Extensibility** | Hard to add new types | Easy to add new types |

---

## Types of Factory Patterns

### 1. Simple Factory (Factory Class) / Practical Factory

**Not a true design pattern, but widely used.**

#### Structure
```
Client → Factory → Products
```

#### Key Characteristic
The **purpose of the class is ONLY to have different methods that return objects of the same class/interface**.

#### Example

```java
// This is a Practical Factory
class PizzaFactory {
    // Only purpose: create Pizza objects
    public Pizza createCheesePizza() { return new CheesePizza(); }
    public Pizza createPepperoniPizza() { return new PepperoniPizza(); }
    public Pizza createVeggiePizza() { return new VeggiePizza(); }
    // NO other business logic - just creates objects!
}
```

#### Visual Representation

```
┌─────────────────────────┐
│   PizzaFactory          │
├─────────────────────────┤
│ + createCheese()        │  ← Returns Pizza
│ + createPepperoni()     │  ← Returns Pizza
│ + createVeggie()        │  ← Returns Pizza
└─────────────────────────┘
     ONLY creates objects!
     No other business logic!
```

---

### 2. Factory Method Pattern

**Uses inheritance. Subclasses decide which class to instantiate.**

#### Structure
```
Client → Creator (abstract) → ConcreteCreator → Products
```

#### Key Characteristic
The **class will have multiple other methods. A few (ideally 1) of those methods might be a method to create an object**.

#### Example

```java
// This is Factory Method Pattern
abstract class PizzaStore {
    // Factory method - just ONE method among many
    abstract Pizza createPizza(String type);
    
    // Other business logic methods:
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);  // Uses factory method
        pizza.prepare();
        pizza.bake();
        return pizza;
    }
    
    public void displayMenu() { /* ... */ }
    public double calculatePrice(Pizza pizza) { /* ... */ }
    public void trackInventory() { /* ... */ }
}
```

#### Visual Representation

```
┌─────────────────────────┐
│   PizzaStore            │
├─────────────────────────┤
│ + createPizza()         │  ← Factory Method
│ + orderPizza()          │  ← Business logic
│ + displayMenu()         │  ← Business logic
│ + calculatePrice()      │  ← Business logic
│ + trackInventory()      │  ← Business logic
└─────────────────────────┘
  Does MORE than just create!
  Factory method is just one part!
```

---

### 3. Abstract Factory Pattern

**Factory of factories. Creates families of related objects.**

#### Structure
```
Client → AbstractFactory → ConcreteFactories → Product Families
```

#### Use Case
When you need to create families of related or dependent objects without specifying their concrete classes.

#### Example Scenario
- GUI Components: Windows UI components vs Mac UI components vs Linux UI components
- Each family has Button, Checkbox, ScrollBar, etc.

---

### Comparison: Practical Factory vs Factory Method

| Aspect | Practical Factory | Factory Method |
|--------|------------------|----------------|
| **Purpose** | ONLY creates objects | Creates objects + has other methods |
| **Class Responsibility** | Single: object creation | Multiple: creation + business logic |
| **Structure** | Simple class with creation methods | Abstract class with factory method |
| **Usage** | Direct instantiation | Inheritance-based |
| **Example** | `PizzaFactory.createCheese()` | `PizzaStore.orderPizza()` which calls `createPizza()` |

---

## Open/Closed Principle & Factory Pattern

### ⚠️ CRITICAL: Does Factory Violate Open/Closed Principle?

**Open/Closed Principle states:**
- **Open** for extension (you can add new features)
- **Closed** for modification (you shouldn't change existing code)

---

### Simple Factory VIOLATES Open/Closed Principle ❌

#### The Problem

```java
class PizzaFactory {
    public Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new CheesePizza();
        } else if (type.equals("pepperoni")) {
            return new PepperoniPizza();
        } else if (type.equals("veggie")) {
            return new VeggiePizza();
        }
        return null;
    }
}
```

**Tomorrow: Boss says "Add Hawaiian Pizza!"**

**What you MUST do:**

```java
class PizzaFactory {
    public Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new CheesePizza();
        } else if (type.equals("pepperoni")) {
            return new PepperoniPizza();
        } else if (type.equals("veggie")) {
            return new VeggiePizza();
        } else if (type.equals("hawaiian")) {    // ← MODIFIED existing class!
            return new HawaiianPizza();          // ← CHANGED existing code!
        }
        return null;
    }
}
```

**Problem:** You had to **MODIFY** the existing factory class!

**Consequences:**
1. ❌ Must change existing, tested code
2. ❌ Risk breaking existing functionality
3. ❌ Need to retest everything
4. ❌ Violates Single Responsibility Principle too

#### Visual Representation

```
┌─────────────────────────┐
│   PizzaFactory          │
├─────────────────────────┤
│ + createPizza(String)   │
│   if "cheese" → new..   │ ← Must modify this code
│   if "pepperoni" → new..│ ← for every new type!
│   if "veggie" → new..   │
│   if "hawaiian" → new.. │ ← Added later (VIOLATION!)
└─────────────────────────┘
```

---

### Solution 1: Factory Method Pattern (Inheritance) ✅

**Follows Open/Closed Principle!**

```java
// ============================================
// Base Creator - NEVER needs modification
// ============================================
abstract class PizzaStore {
    // Factory method
    abstract Pizza createPizza(String type);
    
    // Template method using factory method
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);
        pizza.prepare();
        pizza.bake();
        return pizza;
    }
}

// ============================================
// Concrete Creators - ADD new ones, don't modify old
// ============================================
class NYPizzaStore extends PizzaStore {
    Pizza createPizza(String type) {
        if (type.equals("cheese")) return new NYCheesePizza();
        if (type.equals("pepperoni")) return new NYPepperoniPizza();
        return null;
    }
}

class ChicagoPizzaStore extends PizzaStore {
    Pizza createPizza(String type) {
        if (type.equals("cheese")) return new ChicagoCheesePizza();
        if (type.equals("pepperoni")) return new ChicagoPepperoniPizza();
        return null;
    }
}

// ============================================
// Want California style? Just EXTEND - don't modify existing code!
// ============================================
class CaliforniaPizzaStore extends PizzaStore {
    Pizza createPizza(String type) {
        if (type.equals("cheese")) return new CaliforniaCheesePizza();
        if (type.equals("veggie")) return new CaliforniaVeggiePizza();
        return null;
    }
}

// Usage
PizzaStore nyStore = new NYPizzaStore();
Pizza pizza = nyStore.orderPizza("cheese");
```

**Why this is better:**
- ✅ **Open for extension**: Add new `CaliforniaPizzaStore` without touching old code
- ✅ **Closed for modification**: `NYPizzaStore` and `ChicagoPizzaStore` never change
- ✅ Each store class has single responsibility
- ✅ Easy to test each store independently

#### Visual Representation

```
     ┌──────────────┐
     │ PizzaStore   │ ← Never changes!
     └──────────────┘
            △
            │ extends
    ┌───────┴───────┬───────────┐
    │               │           │
┌─────────┐   ┌──────────┐  ┌─────────────┐
│NYStore  │   │ChicagoS  │  │CaliforniaS  │ ← Just add new!
└─────────┘   └──────────┘  └─────────────┘
```

---


### When to Use Which Solution?

#### Use Simple Factory (despite OCP violation) when:
- ✅ Small project with few types (3-5)
- ✅ Types rarely change
- ✅ Simplicity is more important than perfect design
- ✅ Team is small and changes are easy to manage

#### Use OCP-Compliant Solutions when:
- ✅ Medium to large projects
- ✅ Types frequently added
- ✅ Multiple developers working
- ✅ Plugin architecture needed
- ✅ Types determined by configuration
- ✅ Need to support extensions without recompiling

---

### Real-World Decision Making

```java
// For a small internal tool with 3 notification types
// Simple Factory is FINE:
class NotificationFactory {
    public static Notification create(String type) {
        switch(type) {
            case "email": return new EmailNotification();
            case "sms": return new SMSNotification();
            case "push": return new PushNotification();
            default: throw new IllegalArgumentException();
        }
    }
}

// For a large enterprise system with 20+ notification types
// and third-party plugins, use Registry Pattern:
class NotificationFactory {
    private static Map<String, Supplier<Notification>> registry = new HashMap<>();
    
    public static void register(String type, Supplier<Notification> creator) {
        registry.put(type, creator);
    }
    
    public static Notification create(String type) {
        return registry.get(type).get();
    }
}
```

---

### Key Takeaways: Open/Closed Principle

1. ✅ **Simple Factory DOES violate Open/Closed Principle** - you must modify it for new types
2. ✅ **Factory Method Pattern solves this** - use inheritance to extend
3. ✅ **Registry Pattern is most flexible** - dynamic registration without modification
4. ✅ **Enum Strategy is type-safe** - but limited to compile-time types
5. ✅ **Choose based on project needs** - don't over-engineer small projects!

---

## Complete Working Examples

### Example 1: Simple Factory - Pizza Shop

```java
// ============================================
// STEP 1: Create Product Interface
// ============================================
interface Pizza {
    void prepare();
    void bake();
    void cut();
    void box();
}

// ============================================
// STEP 2: Create Concrete Products
// ============================================
class CheesePizza implements Pizza {
    public void prepare() {
        System.out.println("Preparing Cheese Pizza");
        System.out.println("Tossing dough...");
        System.out.println("Adding tomato sauce...");
        System.out.println("Adding mozzarella cheese...");
    }
    
    public void bake() {
        System.out.println("Baking at 350°F for 15 minutes");
    }
    
    public void cut() {
        System.out.println("Cutting into 8 slices");
    }
    
    public void box() {
        System.out.println("Placing in official pizza box");
    }
}

class PepperoniPizza implements Pizza {
    public void prepare() {
        System.out.println("Preparing Pepperoni Pizza");
        System.out.println("Tossing dough...");
        System.out.println("Adding tomato sauce...");
        System.out.println("Adding cheese and pepperoni...");
    }
    
    public void bake() {
        System.out.println("Baking at 350°F for 18 minutes");
    }
    
    public void cut() {
        System.out.println("Cutting into 8 slices");
    }
    
    public void box() {
        System.out.println("Placing in official pizza box");
    }
}

class VeggiePizza implements Pizza {
    public void prepare() {
        System.out.println("Preparing Veggie Pizza");
        System.out.println("Tossing dough...");
        System.out.println("Adding tomato sauce...");
        System.out.println("Adding vegetables: peppers, onions, mushrooms...");
    }
    
    public void bake() {
        System.out.println("Baking at 350°F for 12 minutes");
    }
    
    public void cut() {
        System.out.println("Cutting into 6 slices");
    }
    
    public void box() {
        System.out.println("Placing in official pizza box");
    }
}

// ============================================
// STEP 3: Create Simple Factory
// ============================================
class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        
        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        
        return pizza;
    }
}

// ============================================
// STEP 4: Create Client (Pizza Store)
// ============================================
class PizzaStore {
    private SimplePizzaFactory factory;
    
    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }
    
    public Pizza orderPizza(String type) {
        Pizza pizza = factory.createPizza(type);
        
        if (pizza != null) {
            System.out.println("\n--- Making a " + type + " pizza ---");
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
            System.out.println("Pizza ready for pickup!\n");
        } else {
            System.out.println("Sorry, we don't make that type of pizza.");
        }
        
        return pizza;
    }
}

// ============================================
// STEP 5: Main Application
// ============================================
public class Main {
    public static void main(String[] args) {
        // Create factory
        SimplePizzaFactory factory = new SimplePizzaFactory();
        
        // Create store with factory
        PizzaStore store = new PizzaStore(factory);
        
        // Order pizzas
        store.orderPizza("cheese");
        store.orderPizza("pepperoni");
        store.orderPizza("veggie");
    }
}
```

**Output:**
```
--- Making a cheese pizza ---
Preparing Cheese Pizza
Tossing dough...
Adding tomato sauce...
Adding mozzarella cheese...
Baking at 350°F for 15 minutes
Cutting into 8 slices
Placing in official pizza box
Pizza ready for pickup!

--- Making a pepperoni pizza ---
Preparing Pepperoni Pizza
Tossing dough...
Adding tomato sauce...
Adding cheese and pepperoni...
Baking at 350°F for 18 minutes
Cutting into 8 slices
Placing in official pizza box
Pizza ready for pickup!

--- Making a veggie pizza ---
Preparing Veggie Pizza
Tossing dough...
Adding tomato sauce...
Adding vegetables: peppers, onions, mushrooms...
Baking at 350°F for 12 minutes
Cutting into 6 slices
Placing in official pizza box
Pizza ready for pickup!
```

---

### Example 2: Factory Method Pattern - Transport System

```java
// ============================================
// STEP 1: Product Interface
// ============================================
interface Transport {
    void deliver(String packageInfo, String destination);
}

// ============================================
// STEP 2: Concrete Products
// ============================================
class Truck implements Transport {
    @Override
    public void deliver(String packageInfo, String destination) {
        System.out.println("\n🚚 TRUCK DELIVERY");
        System.out.println("Package: " + packageInfo);
        System.out.println("Destination: " + destination);
        System.out.println("Route: Highway");
        System.out.println("Estimated delivery: 2 days");
        System.out.println("Status: In transit by road");
    }
}

class Ship implements Transport {
    @Override
    public void deliver(String packageInfo, String destination) {
        System.out.println("\n🚢 SHIP DELIVERY");
        System.out.println("Package: " + packageInfo);
        System.out.println("Destination: " + destination);
        System.out.println("Route: Sea");
        System.out.println("Estimated delivery: 7 days");
        System.out.println("Status: Cargo loaded on vessel");
    }
}

class Plane implements Transport {
    @Override
    public void deliver(String packageInfo, String destination) {
        System.out.println("\n✈️ PLANE DELIVERY");
        System.out.println("Package: " + packageInfo);
        System.out.println("Destination: " + destination);
        System.out.println("Route: Air");
        System.out.println("Estimated delivery: 1 day");
        System.out.println("Status: Loaded on flight");
    }
}

// ============================================
// STEP 3: Creator (Abstract Factory)
// ============================================
abstract class Logistics {
    // Factory method
    public abstract Transport createTransport();
    
    // Template method using factory method
    public void planDelivery(String packageInfo, String destination) {
        Transport transport = createTransport();
        transport.deliver(packageInfo, destination);
    }
}

// ============================================
// STEP 4: Concrete Creators
// ============================================
class RoadLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Ship();
    }
}

class AirLogistics extends Logistics {
    @Override
    public Transport createTransport() {
        return new Plane();
    }
}

// ============================================
// STEP 5: Client Code
// ============================================
public class Main {
    public static void main(String[] args) {
        // Client code works with instances through abstract interface
        Logistics roadLogistics = new RoadLogistics();
        Logistics seaLogistics = new SeaLogistics();
        Logistics airLogistics = new AirLogistics();
        
        // Plan deliveries
        roadLogistics.planDelivery("Furniture Set", "New York");
        seaLogistics.planDelivery("Container of Electronics", "London");
        airLogistics.planDelivery("Urgent Medical Supplies", "Tokyo");
    }
}
```

**Output:**
```
🚚 TRUCK DELIVERY
Package: Furniture Set
Destination: New York
Route: Highway
Estimated delivery: 2 days
Status: In transit by road

🚢 SHIP DELIVERY
Package: Container of Electronics
Destination: London
Route: Sea
Estimated delivery: 7 days
Status: Cargo loaded on vessel

✈️ PLANE DELIVERY
Package: Urgent Medical Supplies
Destination: Tokyo
Route: Air
Estimated delivery: 1 day
Status: Loaded on flight
```

---

### Example 3: Database Connection Factory

```java
// ============================================
// STEP 1: Product Interface
// ============================================
interface Database {
    void connect();
    void executeQuery(String query);
    void disconnect();
}

// ============================================
// STEP 2: Concrete Products
// ============================================
class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("\n[MySQL] Connecting to MySQL database...");
        System.out.println("[MySQL] Connected on port 3306");
        System.out.println("[MySQL] Using InnoDB engine");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("[MySQL] Executing query: " + query);
        System.out.println("[MySQL] Query executed successfully");
    }
    
    @Override
    public void disconnect() {
        System.out.println("[MySQL] Closing connection...");
        System.out.println("[MySQL] Disconnected\n");
    }
}

class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("\n[PostgreSQL] Connecting to PostgreSQL database...");
        System.out.println("[PostgreSQL] Connected on port 5432");
        System.out.println("[PostgreSQL] Using advanced features");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("[PostgreSQL] Executing query: " + query);
        System.out.println("[PostgreSQL] Query executed with ACID compliance");
    }
    
    @Override
    public void disconnect() {
        System.out.println("[PostgreSQL] Closing connection...");
        System.out.println("[PostgreSQL] Disconnected\n");
    }
}

class MongoDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("\n[MongoDB] Connecting to MongoDB...");
        System.out.println("[MongoDB] Connected on port 27017");
        System.out.println("[MongoDB] NoSQL database ready");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("[MongoDB] Executing query: " + query);
        System.out.println("[MongoDB] Document operation completed");
    }
    
    @Override
    public void disconnect() {
        System.out.println("[MongoDB] Closing connection...");
        System.out.println("[MongoDB] Disconnected\n");
    }
}

// ============================================
// STEP 3: Factory
// ============================================
class DatabaseFactory {
    public static Database getDatabase(String dbType) {
        if (dbType == null || dbType.isEmpty()) {
            throw new IllegalArgumentException("Database type cannot be null or empty");
        }
        
        switch(dbType.toUpperCase()) {
            case "MYSQL":
                return new MySQLDatabase();
            case "POSTGRESQL":
            case "POSTGRES":
                return new PostgreSQLDatabase();
            case "MONGODB":
            case "MONGO":
                return new MongoDatabase();
            default:
                throw new IllegalArgumentException("Unknown database type: " + dbType);
        }
    }
}

// ============================================
// STEP 4: Client Application
// ============================================
class DataService {
    private Database database;
    
    public void initializeDatabase(String dbType) {
        try {
            database = DatabaseFactory.getDatabase(dbType);
            database.connect();
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
    
    public void performOperations() {
        if (database != null) {
            database.executeQuery("SELECT * FROM users");
            database.executeQuery("INSERT INTO users VALUES (1, 'John', 'john@example.com')");
            database.executeQuery("UPDATE users SET name='Jane' WHERE id=1");
            database.disconnect();
        }
    }
}

// ============================================
// STEP 5: Main
// ============================================
public class Main {
    public static void main(String[] args) {
        DataService service = new DataService();
        
        System.out.println("===== Testing MySQL =====");
        service.initializeDatabase("MYSQL");
        service.performOperations();
        
        System.out.println("===== Testing PostgreSQL =====");
        service.initializeDatabase("POSTGRESQL");
        service.performOperations();
        
        System.out.println("===== Testing MongoDB =====");
        service.initializeDatabase("MONGO");
        service.performOperations();
    }
}
```

---

### Example 4: Notification System

```java
// ============================================
// Product Interface
// ============================================
interface Notification {
    void send(String recipient, String message);
}

// ============================================
// Concrete Products
// ============================================
class EmailNotification implements Notification {
    @Override
    public void send(String recipient, String message) {
        System.out.println("\n📧 EMAIL NOTIFICATION");
        System.out.println("To: " + recipient);
        System.out.println("Subject: Important Message");
        System.out.println("Body: " + message);
        System.out.println("Status: Email sent successfully");
    }
}

class SMSNotification implements Notification {
    @Override
    public void send(String recipient, String message) {
        System.out.println("\n📱 SMS NOTIFICATION");
        System.out.println("To: " + recipient);
        System.out.println("Message: " + message);
        System.out.println("Status: SMS delivered");
    }
}

class PushNotification implements Notification {
    @Override
    public void send(String recipient, String message) {
        System.out.println("\n🔔 PUSH NOTIFICATION");
        System.out.println("User: " + recipient);
        System.out.println("Alert: " + message);
        System.out.println("Status: Push notification sent to device");
    }
}

class SlackNotification implements Notification {
    @Override
    public void send(String recipient, String message) {
        System.out.println("\n💬 SLACK NOTIFICATION");
        System.out.println("Channel: " + recipient);
        System.out.println("Message: " + message);
        System.out.println("Status: Posted to Slack");
    }
}

// ============================================
// Factory
// ============================================
class NotificationFactory {
    public static Notification createNotification(String channel) {
        switch(channel.toLowerCase()) {
            case "email":
                return new EmailNotification();
            case "sms":
                return new SMSNotification();
            case "push":
                return new PushNotification();
            case "slack":
                return new SlackNotification();
            default:
                throw new IllegalArgumentException("Unsupported notification channel: " + channel);
        }
    }
}

// ============================================
// Service using Factory
// ============================================
class NotificationService {
    public void notifyUser(String channel, String recipient, String message) {
        try {
            Notification notification = NotificationFactory.createNotification(channel);
            notification.send(recipient, message);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
    
    public void notifyMultipleChannels(String recipient, String message, String... channels) {
        System.out.println("\n========== MULTI-CHANNEL NOTIFICATION ==========");
        for (String channel : channels) {
            notifyUser(channel, recipient, message);
        }
        System.out.println("================================================\n");
    }
}

// ============================================
// Main
// ============================================
public class Main {
    public static void main(String[] args) {
        NotificationService service = new NotificationService();
        
        // Single channel notifications
        service.notifyUser("email", "user@example.com", "Your order has been shipped!");
        service.notifyUser("sms", "+1-555-1234", "Your OTP is 123456");
        service.notifyUser("push", "user123", "You have a new message");
        service.notifyUser("slack", "#general", "Deployment completed successfully");
        
        // Multi-channel notification
        service.notifyMultipleChannels(
            "admin@example.com",
            "System alert: High CPU usage detected",
            "email", "sms", "slack"
        );
    }
}
```

---

## When to Use Factory Pattern

### ✅ Use Factory When:

1. **You don't know the exact types beforehand**
   - Object type determined at runtime
   - Type comes from user input or configuration

2. **Object creation is complex**
   - Multiple initialization steps
   - Validation required
   - Dependencies need to be injected

3. **You want to centralize creation logic**
   - Single place to change creation process
   - Easier maintenance

4. **You want loose coupling**
   - Client code doesn't depend on concrete classes
   - Easy to swap implementations

5. **You need to reuse objects**
   - Factory can implement object pooling
   - Control object lifecycle

6. **Creation logic is duplicated**
   - Same object creation code in multiple places
   - Need consistency across codebase

7. **You want better testability**
   - Need to mock objects for testing
   - Want to test without dependencies

### ❌ Don't Use Factory When:

1. **Simple object creation**
   - Just `new MyClass()` is sufficient
   - No complex initialization

2. **Only one type exists**
   - No need for abstraction
   - YAGNI (You Aren't Gonna Need It)

3. **Types never change**
   - No variation needed
   - Over-engineering

4. **Adding unnecessary complexity**
   - Small project with 1-2 simple classes
   - Team is unfamiliar with patterns

---

## Advantages & Disadvantages

### ✅ Advantages

| Advantage | Description | Example |
|-----------|-------------|---------|
| **Loose Coupling** | Client code doesn't depend on concrete classes | Client uses `Transport` interface, not `Truck` |
| **Single Responsibility** | Creation logic separated from business logic | `OrderService` processes orders, `PaymentFactory` creates payments |
| **Open/Closed Principle** | Can add new types without modifying existing code | Add new pizza without changing `PizzaStore` |
| **Encapsulation** | Complex creation logic hidden from clients | Client doesn't see database connection details |
| **Reusability** | Factory can be reused across application | Same `DatabaseFactory` used everywhere |
| **Testability** | Easy to mock and test | Mock factory returns test objects |
| **Flexibility** | Easy to swap implementations | Switch from MySQL to PostgreSQL easily |
| **Centralization** | All creation logic in one place | Change connection string once, affects all |

### ❌ Disadvantages

| Disadvantage | Description | Example |
|--------------|-------------|---------|
| **Complexity** | Adds extra classes and interfaces | Need `Factory`, `Product`, `ConcreteProduct` |
| **Simple Factory violates OCP** | Need to modify factory for new types | Add if-else for each new type |
| **Overhead** | May be overkill for simple scenarios | One class doesn't need factory |
| **Learning Curve** | Requires understanding of abstraction | New developers need to learn pattern |
| **Indirection** | Extra layer between client and objects | `client → factory → product` instead of `client → product` |

---

## Best Practices

### 1. Use Enums for Type Safety

```java
enum TransportType {
    TRUCK, SHIP, PLANE
}

class TransportFactory {
    public static Transport createTransport(TransportType type) {
        switch(type) {
            case TRUCK: return new Truck();
            case SHIP: return new Ship();
            case PLANE: return new Plane();
            default: throw new IllegalArgumentException();
        }
    }
}

// Usage - Type-safe!
Transport t = TransportFactory.createTransport(TransportType.TRUCK);
// TransportFactory.createTransport("TRUK"); // Compile error!
```

### 2. Handle Invalid Input Gracefully

```java
class PizzaFactory {
    public static Pizza createPizza(String type) {
        // Validate input
        if (type == null || type.trim().isEmpty()) {
            throw new IllegalArgumentException("Pizza type cannot be null or empty");
        }
        
        // Create object
        switch(type.toLowerCase()) {
            case "cheese": return new CheesePizza();
            case "pepperoni": return new PepperoniPizza();
            default: 
                throw new IllegalArgumentException("Unknown pizza type: " + type);
        }
    }
}
```

### 3. Use Static Factory Methods with Meaningful Names

```java
class Database {
    private String host;
    private int port;
    
    private Database(String host, int port) {
        this.host = host;
        this.port = port;
    }
    
    // Static factory methods with meaningful names
    public static Database mysql() {
        return new Database("localhost", 3306);
    }
    
    public static Database postgres() {
        return new Database("localhost", 5432);
    }
    
    public static Database custom(String host, int port) {
        return new Database(host, port);
    }
}

// Usage - Clear and readable!
Database db1 = Database.mysql();
Database db2 = Database.postgres();
Database db3 = Database.custom("192.168.1.1", 5432);
```

### 4. Consider Using Registry Pattern for Extensibility

```java
class ShapeFactory {
    private static Map<String, Supplier<Shape>> registry = new HashMap<>();
    
    // Register shape creators
    static {
        registry.put("circle", Circle::new);
        registry.put("square", Square::new);
        registry.put("triangle", Triangle::new);
    }
    
    // Allow external registration
    public static void register(String type, Supplier<Shape> creator) {
        registry.put(type.toLowerCase(), creator);
    }
    
    public static Shape createShape(String type) {
        Supplier<Shape> creator = registry.get(type.toLowerCase());
        if (creator == null) {
            throw new IllegalArgumentException("Unknown shape: " + type);
        }
        return creator.get();
    }
}

// Add new shape without modifying factory
ShapeFactory.register("hexagon", Hexagon::new);
Shape hexagon = ShapeFactory.createShape("hexagon");
```

### 5. Return Interfaces, Not Concrete Classes

```java
// ✅ Good - Returns interface
public static Transport createTransport(String type) {
    return new Truck();  // Returns Transport interface
}

// ❌ Bad - Returns concrete class
public static Truck createTruck() {
    return new Truck();  // Exposes concrete class
}
```

### 6. Use Dependency Injection with Factories

```java
// Factory interface
interface PaymentFactory {
    Payment createPayment(Order order);
}

// Client depends on factory interface
class OrderService {
    private PaymentFactory paymentFactory;
    
    // Inject factory via constructor
    public OrderService(PaymentFactory paymentFactory) {
        this.paymentFactory = paymentFactory;
    }
    
    public void processOrder(Order order) {
        Payment payment = paymentFactory.createPayment(order);
        payment.process();
    }
}

// Easy to swap factories or mock for testing
OrderService service = new OrderService(new CreditCardPaymentFactory());
```

### 7. Document Factory Behavior Clearly

```java
/**
 * Creates notification objects based on channel type.
 * 
 * @param channel The notification channel ("email", "sms", "push", "slack")
 * @return Notification instance for the specified channel
 * @throws IllegalArgumentException if channel is null, empty, or unsupported
 * 
 * Supported channels:
 * - "email": Returns EmailNotification
 * - "sms": Returns SMSNotification  
 * - "push": Returns PushNotification
 * - "slack": Returns SlackNotification
 */
public static Notification createNotification(String channel) {
    // Implementation
}
```

---

## Real-World Use Cases

### 1. JDBC Database Drivers

```java
// Factory Method Pattern in action
Connection conn = DriverManager.getConnection(url, user, password);
// Factory creates appropriate driver: MySQL, PostgreSQL, Oracle, etc.
```

### 2. Java Collections Framework

```java
// Static factory methods
List<String> emptyList = Collections.emptyList();
List<String> singletonList = Collections.singletonList("item");
List<String> unmodifiableList = Collections.unmodifiableList(myList);
// Factory methods create appropriate list implementations
```

### 3. Spring Framework

```java
@Bean
public DataSource dataSource() {
    return DataSourceBuilder.create()
        .driverClassName("com.mysql.jdbc.Driver")
        .url("jdbc:mysql://localhost:3306/mydb")
        .username("user")
        .password("password")
        .build();
}
// Factory creates appropriate DataSource implementation
```

### 4. Logger Frameworks

```java
// SLF4J uses Factory Pattern
Logger logger = LoggerFactory.getLogger(MyClass.class);
// Factory creates appropriate logger: Log4j, Logback, JUL, etc.
```

### 5. Document Parsers

```java
// Factory creates XML parser
DocumentBuilder builder = DocumentBuilderFactory.newInstance()
    .newDocumentBuilder();
Document doc = builder.parse(new File("data.xml"));
```

### 6. GUI Components

```java
// Factory creates platform-specific components
Button button = UIComponentFactory.createButton("windows");
// Creates Windows, Mac, or Linux specific button
```

### 7. Payment Gateways

```java
// Factory creates appropriate payment processor
PaymentProcessor processor = PaymentFactory.createProcessor(paymentMethod);
processor.process(amount);
// Creates Stripe, PayPal, Square, etc.
```

### 8. Cloud Storage Providers

```java
// Factory creates storage client
StorageClient client = StorageFactory.createClient(provider);
client.upload(file);
// Creates AWS S3, Google Cloud, Azure, Dropbox clients
```

---

## Quick Reference Guide

### Factory Pattern Decision Tree

```
Need to create objects?
│
├─ Few types (3-5) that rarely change?
│  └─ Use Simple Factory (if-else is fine)
│
├─ Multiple product families (NY, Chicago, etc.)?
│  └─ Use Factory Method Pattern
│
├─ Need plugin system or dynamic types?
│  └─ Use Registry Pattern
│
├─ Fixed set of types, need type safety?
│  └─ Use Enum Strategy
│
└─ Very large system with 50+ types?
   └─ Use Annotation-based with auto-registration
```

---

### Code Templates

#### Simple Factory Template

```java
class ProductFactory {
    public static Product createProduct(String type) {
        switch(type) {
            case "typeA": return new ProductA();
            case "typeB": return new ProductB();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}
```

#### Factory Method Template

```java
abstract class Creator {
    // Factory method
    abstract Product createProduct();
    
    // Template method using factory method
    public void operation() {
        Product p = createProduct();
        // use product
    }
}

class ConcreteCreator extends Creator {
    @Override
    Product createProduct() {
        return new ConcreteProduct();
    }
}
```

#### Registry Pattern Template

```java
class Factory {
    private static Map<String, Supplier<Product>> registry = new HashMap<>();
    
    public static void register(String type, Supplier<Product> creator) {
        registry.put(type, creator);
    }
    
    public static Product create(String type) {
        Supplier<Product> creator = registry.get(type);
        if (creator == null) {
            throw new IllegalArgumentException("Unknown type: " + type);
        }
        return creator.get();
    }
}
```

---

### Comparison Table

| Feature | Simple Factory | Factory Method | Abstract Factory |
|---------|---------------|----------------|------------------|
| **Complexity** | Low | Medium | High |
| **Classes** | 1 factory class | Multiple creator subclasses | Multiple factory classes |
| **Extensibility** | Modify factory | Extend creator | Extend factory family |
| **Use Case** | Single product family | Subclass decides product | Multiple related products |
| **OCP Compliant** | ❌ No | ✅ Yes | ✅ Yes |
| **Example** | Pizza factory | Transport logistics | UI component themes |
| **Best For** | Small projects | Medium projects | Large systems |

---

### UML Diagrams

#### Simple Factory

```
┌─────────┐         ┌─────────────────┐
│ Client  │────────>│ SimpleFactory   │
└─────────┘         │ + create()      │
                    └─────────────────┘
                            │
                            ↓ creates
                    ┌───────────────┐
                    │   Product     │
                    ├───────────────┤
                    │ ConcreteA     │
                    │ ConcreteB     │
                    └───────────────┘
```

#### Factory Method

```
┌─────────┐         ┌──────────────────┐
│ Client  │────────>│ Creator          │
└─────────┘         │ + factoryMethod()│
                    │ + operation()    │
                    └──────────────────┘
                            △
                            │
                ┌───────────┴───────────┐
                │                       │
        ┌───────────────┐       ┌───────────────┐
        │ ConcreteA     │       │ ConcreteB     │
        │ + factoryM()  │       │ + factoryM()  │
        └───────────────┘       └───────────────┘
```

---

### Real-World Analogy

#### Without Factory (Making Coffee Yourself)

```
You want coffee...
↓
You must:
1. Buy coffee beans
2. Grind beans
3. Heat water
4. Brew coffee
5. Add milk/sugar
6. Clean up

Problem: Every time you want coffee, you do ALL steps yourself!
```

#### With Factory (Coffee Shop / Barista)

```
You want coffee...
↓
You tell barista: "I want a latte"
↓
Barista (Factory) handles everything
↓
You get your coffee

Benefit: You don't need to know how to make coffee!
         Barista handles all complexity!
```

**Key Insight:** Factory Pattern is like having a specialized coffee shop (barista) instead of making coffee yourself every time!

---

### Summary Cheat Sheet

#### Core Definition
**Factory Pattern** = Delegate object creation to a dedicated factory, instead of creating objects directly in your code.

#### Problems It Solves (Quick List)
1. ✅ Removes tight coupling between client code and concrete classes
2. ✅ Enforces Single Responsibility by separating creation from usage
3. ✅ Eliminates code duplication by centralizing creation logic
4. ✅ Encapsulates complexity of object creation
5. ✅ Enables easy switching of implementations
6. ✅ Improves testability by allowing mock objects
7. ✅ Centralizes conditional logic for object creation

#### When to Use (Quick Guide)
**Use Factory Pattern when:**
- ✅ You have multiple similar classes
- ✅ Object creation is complex
- ✅ You want to reduce coupling
- ✅ You need to switch implementations
- ✅ Creation logic is duplicated
- ✅ You want better testability

**Don't use when:**
- ❌ Only one simple class
- ❌ Object creation is trivial (`new MyClass()`)
- ❌ Types never change
- ❌ Would add unnecessary complexity

---

### Practice Exercises

#### Exercise 1: Coffee Shop
Create a factory for different coffee types (Espresso, Latte, Cappuccino, Americano) with different brewing methods and prices.

#### Exercise 2: Payment Gateway
Implement a factory for payment methods (CreditCard, PayPal, Stripe, Bitcoin) with different processing logic and transaction fees.

#### Exercise 3: File Parser
Create a factory that returns different parsers (CSV, JSON, XML, Excel) based on file extension, each with appropriate parsing logic.

#### Exercise 4: Cloud Storage
Implement a factory for cloud storage providers (AWS S3, Google Cloud, Azure, Dropbox) with different APIs for upload/download.

---

### Final Thoughts

> "The Factory Pattern isn't about making code more complicated. It's about making code **easier to change, easier to test, and easier to understand** by separating the 'what' (business logic) from the 'how' (object creation)."

**Remember:**
- Start simple - use Simple Factory for small projects
- Refactor to Factory Method or Registry when you need extensibility
- Don't over-engineer - choose the right solution for your project size
- The goal is **maintainability**, not complexity

---
