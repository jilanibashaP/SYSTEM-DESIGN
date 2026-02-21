# LLD Interview Approach — CERID Framework

> **Don't start coding. Follow CERID.**

We'll use **three running examples** throughout:
- 🔔 **Notification System**
- 🅿️ **Parking Lot**
- 🛒 **Shopping Cart / Order System**

---

## C — Clarify (Requirements)

**Goal:** Remove ambiguity, set functional and non-functional requirements *before* designing anything.

Ask yourself:
- What are the core features?
- What operations must the system support?
- Any constraints (scale, concurrency, persistence)?
- What is explicitly **out of scope**?

### 🔔 Notification System
| Question | Clarification |
|----------|--------------|
| Core features? | Send notifications via Email, SMS, Push |
| Operations? | `sendNotification(userId, message, type)` |
| Constraints? | High throughput, retry on failure |
| Out of scope? | Billing, user authentication |

### 🅿️ Parking Lot
| Question | Clarification |
|----------|--------------|
| Core features? | Park vehicles, free spots, calculate fee |
| Operations? | `parkVehicle()`, `removeVehicle()`, `getAvailableSpots()` |
| Constraints? | Multiple floors, multiple vehicle types |
| Out of scope? | Online booking, subscriptions |

### 🛒 Shopping Cart
| Question | Clarification |
|----------|--------------|
| Core features? | Add/remove items, place order, apply discount |
| Operations? | `addItem()`, `removeItem()`, `checkout()` |
| Constraints? | Concurrent users, inventory consistency |
| Out of scope? | Payment gateway integration, shipping tracking |

---

## E — Entities

**Goal:** Define the structural backbone of the system.

- Extract key **nouns** from the problem statement.
- These usually become your **primary classes**.
- Avoid overthinking — just identify major building blocks.

### 🔔 Notification System
```
User, Notification, NotificationChannel, NotificationService
```

### 🅿️ Parking Lot
```
ParkingLot, Floor, ParkingSpot, Vehicle, Ticket, FeeCalculator
```

### 🛒 Shopping Cart
```
User, Cart, CartItem, Product, Order, Discount, Inventory
```

> **Tip:** If you can underline it in the problem statement, it's probably an entity.

---

## R — Responsibilities

**Goal:** Avoid god classes and messy logic distribution.

For each entity, define:
- What does it **own**?
- What **behavior** does it handle?
- What should it **NOT** handle?

Follow:
- **Single Responsibility Principle** — one reason to change
- **High cohesion** — related things stay together
- **Low coupling** — classes don't depend on each other's internals

### 🔔 Notification System

| Class | Owns | Handles | Does NOT handle |
|-------|------|---------|-----------------|
| `User` | contact info, preferences | get/set channel prefs | sending messages |
| `Notification` | message, type, timestamp | hold data | delivery logic |
| `NotificationChannel` | channel config | `send(notification)` | user lookup |
| `NotificationService` | channel registry | orchestrate sending, retry | message content |

### 🅿️ Parking Lot

| Class | Owns | Handles | Does NOT handle |
|-------|------|---------|-----------------|
| `ParkingSpot` | spotId, type, isOccupied | `assignVehicle()`, `free()` | fee calculation |
| `Vehicle` | licensePlate, type | hold vehicle data | spot selection |
| `Ticket` | entry time, spot, vehicle | hold ticket data | payment processing |
| `FeeCalculator` | rate config | `calculateFee(ticket)` | issuing tickets |
| `ParkingLot` | floors, spots | `findSpot()`, `issueTicket()` | fee logic |

### 🛒 Shopping Cart

| Class | Owns | Handles | Does NOT handle |
|-------|------|---------|-----------------|
| `Cart` | list of CartItems | `addItem()`, `removeItem()`, `getTotal()` | order creation |
| `CartItem` | product, quantity | hold item data | pricing rules |
| `Order` | items snapshot, status | `place()`, `cancel()` | cart management |
| `Discount` | discount rules | `apply(cart)` | inventory check |
| `Inventory` | stock counts | `reserve()`, `release()` | pricing |

---

## I — Interactions (Relationships)

**Goal:** Understand how the system *behaves*, not just how it looks.

Connect the entities — think in terms of use cases:
> *"When X happens, which objects collaborate?"*

- **Composition** — child cannot exist without parent (e.g. `Cart` owns `CartItems`)
- **Association** — independent objects refer to each other (e.g. `Order` references `User`)
- **Dependency** — one uses another but doesn't own it (e.g. `Service` uses `Channel` interface)

---

### 🔔 Notification System — Flow: "Send a notification"

```
Client
  └──▶ NotificationService.send(userId, message, type)
            ├── UserRepository.getUser(userId)        → User (preferences)
            ├── ChannelSelector.select(user.prefs)    → [EmailChannel, SMSChannel]
            ├── Notification.create(message, type)    → Notification
            └── for each Channel:
                    Channel.send(notification)
                    └── on failure → RetryPolicy.retry()
```

**Relationships:**
- `NotificationService` → *depends on* `Channel` interface (not concrete classes)
- `User` → *owns* `Preferences` (composition)
- `EmailChannel`, `SMSChannel`, `PushChannel` → *independent implementations* of `Channel`

---

### 🅿️ Parking Lot — Flow: "Park a vehicle"

```
Driver arrives
  └──▶ ParkingLot.parkVehicle(vehicle)
            ├── SpotFinder.findAvailableSpot(vehicle.type)  → ParkingSpot
            ├── ParkingSpot.assignVehicle(vehicle)
            └── TicketIssuer.issueTicket(spot, vehicle)     → Ticket (with entry time)

Driver leaves
  └──▶ ParkingLot.removeVehicle(ticket)
            ├── FeeCalculator.calculateFee(ticket)          → amount
            ├── ParkingSpot.free()
            └── return amount to driver
```

**Relationships:**
- `ParkingLot` *has* many `Floor`s (composition)
- `Floor` *has* many `ParkingSpot`s (composition)
- `Ticket` *references* `ParkingSpot` and `Vehicle` (association)
- `FeeCalculator` is *independent* — can swap strategies (e.g. hourly vs. flat rate)

---

### 🛒 Shopping Cart — Flow: "Checkout"

```
User clicks Checkout
  └──▶ CheckoutService.checkout(cartId, userId)
            ├── Cart.getItems()                          → [CartItem, ...]
            ├── Inventory.reserve(items)                 → success / failure
            ├── Discount.apply(cart)                     → discounted total
            ├── Order.create(cart, user)                 → Order
            └── Cart.clear()
```

**Relationships:**
- `Cart` *owns* `CartItem`s (composition — CartItem dies with Cart)
- `Order` *references* `User` and snapshot of items (association)
- `CheckoutService` *depends on* `Inventory`, `Discount`, `OrderRepository` (dependency injection)

---

## D — Durability (Easy to incorporate changes)

**Goal:** Design for change, not just current requirements.

Before coding, ask:
- What might **change** in the future?
- Should I use **interfaces**?
- Do I need a **design pattern** here?
- Am I **tightly coupling** anything?

---

### 🔔 Notification System

| Future change | Solution |
|--------------|----------|
| Add WhatsApp channel | `Channel` interface → just add `WhatsAppChannel implements Channel` |
| Add scheduled notifications | `NotificationScheduler` wraps `NotificationService` |
| Priority-based delivery | Strategy pattern for `ChannelSelector` |
| Retry logic changes | Encapsulate in `RetryPolicy` — swap without touching `Service` |

```java
// ✅ Durable — depend on interface, not concrete class
interface NotificationChannel {
    void send(Notification notification);
}

class EmailChannel implements NotificationChannel { ... }
class SMSChannel implements NotificationChannel { ... }
class WhatsAppChannel implements NotificationChannel { ... }  // add tomorrow, no changes elsewhere!
```

### 🅿️ Parking Lot

| Future change | Solution |
|--------------|----------|
| Add EV charging spots | Extend `ParkingSpot` with `EVSpot` subclass |
| Add dynamic pricing | `FeeCalculator` is already isolated — swap strategy |
| Add online booking | New `Reservation` entity + `BookingService`, doesn't break existing code |
| Multi-city parking lots | `ParkingLot` already abstracted — wrap in `ParkingLotManager` |

```java
// ✅ Durable — fee strategy is swappable
interface FeeCalculator {
    double calculate(Ticket ticket);
}

class HourlyFeeCalculator implements FeeCalculator { ... }
class FlatRateFeeCalculator implements FeeCalculator { ... }  // add without touching ParkingLot
```

### 🛒 Shopping Cart

| Future change | Solution |
|--------------|----------|
| Add new discount type | `Discount` interface → add `BuyOneGetOneDiscount` without changing `Cart` |
| Add new payment method | `PaymentGateway` interface → plug in new provider |
| Add wishlist feature | New `Wishlist` entity — `Cart` stays untouched |
| Cart persistence | Extract `CartRepository` — swap in-memory for DB |

```java
// ✅ Durable — discounts are pluggable
interface Discount {
    double apply(Cart cart);
}

class PercentageDiscount implements Discount { ... }
class CouponDiscount implements Discount { ... }
class BuyOneGetOneDiscount implements Discount { ... }  // new requirement, no Cart changes
```

---

## ⭐ Bonus: Thread Safety

**Always build the system which is thread safe.**

How to check for thread safety:
1. Check with interviewer if **multiple threads** will be accessing the system.
2. Is there **shared mutable state**?
3. Do I need **synchronization**?
4. Should I use **concurrent collections**?

### 🔔 Notification System
```java
// ❌ Not thread safe — multiple threads could modify channel list simultaneously
List<NotificationChannel> channels = new ArrayList<>();

// ✅ Thread safe
List<NotificationChannel> channels = new CopyOnWriteArrayList<>();
// CopyOnWriteArrayList is best when reads >> writes (channel list rarely changes)
```

### 🅿️ Parking Lot
```java
// ❌ Not thread safe — two cars could get assigned the same spot (race condition!)
public ParkingSpot findAvailableSpot() {
    for (ParkingSpot spot : spots) {
        if (!spot.isOccupied()) return spot;  // another thread grabs it right here!
    }
}

// ✅ Thread safe — synchronize the critical find-and-assign operation atomically
public synchronized ParkingSpot findAndAssignSpot(Vehicle vehicle) {
    for (ParkingSpot spot : spots) {
        if (!spot.isOccupied()) {
            spot.assignVehicle(vehicle);  // find + assign in one atomic step
            return spot;
        }
    }
    return null;
}
```

### 🛒 Shopping Cart
```java
// ❌ Not thread safe — two users could buy the last item simultaneously
int stock = inventory.get(productId);      // thread A reads stock = 1
                                           // thread B also reads stock = 1
inventory.put(productId, stock - qty);     // both set stock to 0 — oversold!

// ✅ Thread safe — atomic compare-and-update
ConcurrentHashMap<String, AtomicInteger> inventory = new ConcurrentHashMap<>();
boolean reserved = inventory.get(productId).compareAndSet(currentStock, currentStock - qty);
// Only one thread wins; the other retries
```

---

## Quick Reference Card

| Step | Focus | Key Question | Common Mistake |
|------|-------|--------------|----------------|
| **C**larify | Requirements | What exactly needs to be built? | Assuming scope without asking |
| **E**ntities | Core classes | What are the major nouns/building blocks? | Jumping to methods too early |
| **R**esponsibilities | Class design | What does each class own and do? | Creating god classes |
| **I**nteractions | Relationships & flow | How do objects collaborate? | Skipping the flow, only drawing boxes |
| **D**urability | Extensibility | What might change and how do I prepare? | Hard-coding concrete classes |
| **Bonus** | Thread safety | Is concurrent access handled correctly? | Forgetting shared mutable state |
