# Database Concurrency Problems - Detailed Explanation

## Table of Contents
1. [What Are Concurrency Problems?](#what-are-concurrency-problems)
2. [Dirty Read](#1-dirty-read)
3. [Non-Repeatable Read](#2-non-repeatable-read)
4. [Phantom Read](#3-phantom-read)
5. [Lost Update](#4-lost-update)
6. [Comparison Table](#comparison-table)
7. [How Isolation Levels Prevent These Problems](#how-isolation-levels-prevent-these-problems)
8. [Real-World Examples](#real-world-examples)

---

## What Are Concurrency Problems?

When multiple transactions run **simultaneously** (concurrently), they can interfere with each other and cause **data inconsistency**. These problems occur when transactions read or write data that other transactions are also accessing.

**Why do these problems happen?**
- Multiple users accessing the same data at the same time
- One transaction reads data while another is modifying it
- Lack of proper isolation between transactions

**The Solution:** Database isolation levels and locking mechanisms prevent these problems.

---

## 1. Dirty Read

### What is a Dirty Read?

A **dirty read** occurs when a transaction reads data that has been **modified but not yet committed** by another transaction. If the modifying transaction rolls back, the data read was never valid.

### Why is it called "Dirty"?

Because you're reading **uncommitted, potentially invalid data** - data that might be "rolled back" and never actually exist in the database.

---

### Example Scenario: Bank Account Transfer

```java
// Initial state: Account balance = $500

// ============================================
// Transaction 1: Attempting to transfer money
// ============================================
START TRANSACTION;
UPDATE account SET balance = 1000 WHERE id = 1;  
// Balance is now $1000 (but NOT committed yet!)

// ============================================
// Transaction 2: Checking balance
// ============================================
START TRANSACTION;
SELECT balance FROM account WHERE id = 1;  
// Reads $1000 (DIRTY READ - uncommitted data!)

// Transaction 2 makes a decision based on $1000 balance
if (balance >= 900) {
    processLoan();  // Loan approved based on dirty data!
}
COMMIT;

// ============================================
// Transaction 1: Something goes wrong
// ============================================
ROLLBACK;  
// Balance returns to $500 (the update never happened!)
```

**The Problem:**
- Transaction 2 saw balance = $1000
- Transaction 1 rolled back, so balance was actually $500 all along
- Transaction 2 approved a loan based on **data that never existed**

---

### Real-World Impact

```java
// E-commerce Example
// Transaction 1: User adds item to cart
START TRANSACTION;
UPDATE inventory SET quantity = 5 WHERE product_id = 101;
// Reduced from 10 to 5 (not committed)

// Transaction 2: Admin checks inventory
SELECT quantity FROM inventory WHERE product_id = 101;
// Sees 5 items (DIRTY READ)
// Sends alert: "Low stock! Only 5 left"

// Transaction 1: User cancels
ROLLBACK;
// Quantity is actually still 10!
```

---

### Code Example in Spring Boot

```java
@Service
public class AccountService {
    
    // Transaction 1: Updates balance (doesn't commit immediately)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public void updateBalance(Long accountId, BigDecimal amount) {
        Account account = accountRepo.findById(accountId).orElseThrow();
        account.setBalance(amount);
        accountRepo.save(account);
        
        // Simulating delay before commit
        Thread.sleep(5000);
        // Not committed yet!
    }
    
    // Transaction 2: Reads uncommitted balance
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public BigDecimal checkBalance(Long accountId) {
        Account account = accountRepo.findById(accountId).orElseThrow();
        return account.getBalance();  // DIRTY READ - might see uncommitted data!
    }
}
```

---

### How to Prevent Dirty Reads

Use **READ_COMMITTED** isolation level or higher:

```java
// ✅ SOLUTION: Use READ_COMMITTED
@Transactional(isolation = Isolation.READ_COMMITTED)
public BigDecimal checkBalance(Long accountId) {
    Account account = accountRepo.findById(accountId).orElseThrow();
    return account.getBalance();  // Only reads committed data
}
```

---

### Visual Timeline

```
Time    Transaction 1                  Transaction 2
------------------------------------------------------------
t1      START TRANSACTION              
t2      UPDATE balance = 1000          
t3                                     START TRANSACTION
t4                                     SELECT balance
t5                                     (reads 1000) ← DIRTY READ!
t6                                     COMMIT
t7      ROLLBACK (back to 500)         
t8      Transaction 2 used wrong data! ❌
```

---

## 2. Non-Repeatable Read

### What is a Non-Repeatable Read?

A **non-repeatable read** occurs when a transaction reads the **same row twice** and gets **different values** because another transaction modified and committed the data between the two reads.

### Why is it called "Non-Repeatable"?

Because if you repeat the same read query within your transaction, you get **different results** - the read cannot be repeated with consistent results.

---

### Example Scenario: Monthly Report Generation

```java
// Initial state: Account balance = $500

// ============================================
// Transaction 1: Generating monthly report
// ============================================
START TRANSACTION;

// First read at beginning of report
SELECT balance FROM account WHERE id = 1;  
// Returns $500

// ... processing other data ...

// ============================================
// Transaction 2: Customer deposits money
// ============================================
START TRANSACTION;
UPDATE account SET balance = 1000 WHERE id = 1;
COMMIT;  // ✅ Committed!

// ============================================
// Transaction 1: Reading again for verification
// ============================================
// Second read at end of report
SELECT balance FROM account WHERE id = 1;  
// Returns $1000 (NON-REPEATABLE READ - different value!)

COMMIT;
```

**The Problem:**
- Transaction 1 read balance as $500
- Transaction 1 read balance again as $1000
- Same query, same transaction, **different results**
- Report has **inconsistent data** ($500 in one place, $1000 in another)

---

### Real-World Impact

```java
// Financial Report Example
@Transactional(isolation = Isolation.READ_COMMITTED)
public FinancialReport generateReport(Long accountId) {
    
    // First read: Get opening balance
    Account account = accountRepo.findById(accountId).orElseThrow();
    BigDecimal openingBalance = account.getBalance();  // $500
    
    // Process transactions for the month
    List<Transaction> transactions = getMonthlyTransactions(accountId);
    BigDecimal totalDebits = calculateDebits(transactions);
    BigDecimal totalCredits = calculateCredits(transactions);
    
    // Another transaction modifies the account and commits here!
    // UPDATE account SET balance = 1000 WHERE id = accountId;
    
    // Second read: Get closing balance
    account = accountRepo.findById(accountId).orElseThrow();
    BigDecimal closingBalance = account.getBalance();  // $1000 (CHANGED!)
    
    // Calculate expected vs actual
    BigDecimal expected = openingBalance.add(totalCredits).subtract(totalDebits);
    // expected might be $800
    // closingBalance is $1000
    // MISMATCH! Report is inconsistent ❌
    
    return new FinancialReport(openingBalance, closingBalance, expected);
}
```

---

### Code Example with Problem

```java
@Service
public class ReportService {
    
    @Autowired
    private AccountRepository accountRepo;
    
    // ❌ PROBLEM: READ_COMMITTED allows non-repeatable reads
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void demonstrateNonRepeatableRead(Long accountId) {
        
        // First read
        Account account1 = accountRepo.findById(accountId).orElseThrow();
        BigDecimal balance1 = account1.getBalance();
        System.out.println("First read: " + balance1);  // e.g., $500
        
        // Simulate processing time
        try {
            Thread.sleep(2000);  // 2 seconds
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // Meanwhile, another transaction updates and commits
        
        // Second read - SAME transaction, SAME query
        Account account2 = accountRepo.findById(accountId).orElseThrow();
        BigDecimal balance2 = account2.getBalance();
        System.out.println("Second read: " + balance2);  // e.g., $1000
        
        // Data changed within same transaction!
        if (!balance1.equals(balance2)) {
            System.out.println("NON-REPEATABLE READ DETECTED! ❌");
            System.out.println("First: " + balance1 + ", Second: " + balance2);
        }
    }
}
```

---

### How to Prevent Non-Repeatable Reads

Use **REPEATABLE_READ** or **SERIALIZABLE** isolation level:

```java
// ✅ SOLUTION: Use REPEATABLE_READ
@Transactional(isolation = Isolation.REPEATABLE_READ)
public FinancialReport generateConsistentReport(Long accountId) {
    
    // First read
    Account account = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance1 = account.getBalance();  // $500
    
    // Database holds a read lock on this row
    // Other transactions CANNOT modify it until we commit
    
    // Processing...
    
    // Second read - GUARANTEED to be same value
    account = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance2 = account.getBalance();  // Still $500 ✅
    
    // balance1 == balance2 (GUARANTEED)
    
    return generateReport(balance1, balance2);
}
```

---

### Visual Timeline

```
Time    Transaction 1                      Transaction 2
----------------------------------------------------------------
t1      START TRANSACTION (READ_COMMITTED)              
t2      SELECT balance (reads $500)        
t3                                         START TRANSACTION
t4                                         UPDATE balance = $1000
t5                                         COMMIT ✅
t6      SELECT balance (reads $1000)       ← NON-REPEATABLE READ!
t7      COMMIT                             
t8      Transaction 1 has inconsistent data ❌

------- WITH REPEATABLE_READ -------

Time    Transaction 1                      Transaction 2
----------------------------------------------------------------
t1      START TRANSACTION (REPEATABLE_READ)             
t2      SELECT balance (reads $500)        
        [Holds read lock on row]
t3                                         START TRANSACTION
t4                                         UPDATE balance = $1000
t5                                         ⏳ WAITING for lock...
t6      SELECT balance (reads $500 again) ✅
t7      COMMIT [releases lock]             
t8                                         ✅ Now can commit
```

---

## 3. Phantom Read

### What is a Phantom Read?

A **phantom read** occurs when a transaction executes the **same query twice** and gets a **different number of rows** because another transaction **inserted or deleted rows** that match the query criteria.

### Why is it called "Phantom"?

Because **new rows appear** (or existing rows disappear) like "phantoms" between two identical queries in the same transaction.

---

### Key Difference from Non-Repeatable Read

| Problem | What Changes | Example |
|---------|-------------|---------|
| **Non-Repeatable Read** | **Existing row values** change | Row 1: balance changes from $500 to $1000 |
| **Phantom Read** | **Number of rows** changes | Query returns 5 rows, then returns 6 rows |

---

### Example Scenario: Generating Account Summary

```java
// Initial state: 5 accounts with balance > $1000

// ============================================
// Transaction 1: Generating high-balance report
// ============================================
START TRANSACTION;

// First query: Find all high-balance accounts
SELECT * FROM account WHERE balance > 1000;  
// Returns 5 rows:
// Account 1: $1500
// Account 2: $2000
// Account 3: $1200
// Account 4: $3000
// Account 5: $1800

int count1 = 5;
BigDecimal total1 = $9500;

// ... processing data ...

// ============================================
// Transaction 2: New customer opens account
// ============================================
START TRANSACTION;
INSERT INTO account (id, balance) VALUES (6, 5000);
COMMIT;  // ✅ New account added!

// ============================================
// Transaction 1: Re-running query for verification
// ============================================
// Second query: Same query, same transaction
SELECT * FROM account WHERE balance > 1000;  
// Returns 6 rows (PHANTOM READ - new row appeared!):
// Account 1: $1500
// Account 2: $2000
// Account 3: $1200
// Account 4: $3000
// Account 5: $1800
// Account 6: $5000  ← PHANTOM ROW!

int count2 = 6;  // count1 != count2
BigDecimal total2 = $14500;  // total1 != total2

COMMIT;
```

**The Problem:**
- Transaction 1 counted 5 accounts with total $9500
- Transaction 1 counted again: 6 accounts with total $14500
- Same query, **different number of rows**
- Report has **inconsistent counts and totals**

---

### Real-World Impact

```java
// Statistical Report Example
@Transactional(isolation = Isolation.REPEATABLE_READ)
public StatisticalReport generateStats(BigDecimal threshold) {
    
    // First query: Count high-balance accounts
    List<Account> accounts1 = accountRepo.findByBalanceGreaterThan(threshold);
    int count1 = accounts1.size();  // e.g., 100 accounts
    BigDecimal avgBalance1 = calculateAverage(accounts1);  // e.g., $5000
    
    // Processing data...
    Map<String, Object> demographics = analyzeDemographics(accounts1);
    
    // Meanwhile, other transactions insert/delete accounts
    // INSERT INTO account (balance) VALUES (10000);
    // INSERT INTO account (balance) VALUES (15000);
    // DELETE FROM account WHERE balance = 5500;
    
    // Second query: Re-count for final report
    List<Account> accounts2 = accountRepo.findByBalanceGreaterThan(threshold);
    int count2 = accounts2.size();  // e.g., 102 accounts (CHANGED!)
    BigDecimal avgBalance2 = calculateAverage(accounts2);  // e.g., $5200 (CHANGED!)
    
    // INCONSISTENCY:
    // - Demographics based on 100 accounts
    // - Final count shows 102 accounts
    // - Average balance different
    // Report is INVALID! ❌
    
    return new StatisticalReport(count2, avgBalance2, demographics);
}
```

---

### Code Example with Problem

```java
@Service
public class AnalyticsService {
    
    @Autowired
    private OrderRepository orderRepo;
    
    // ❌ PROBLEM: Even REPEATABLE_READ allows phantom reads
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void demonstratePhantomRead(String status, LocalDate date) {
        
        // First query: Count orders
        List<Order> orders1 = orderRepo.findByStatusAndDate(status, date);
        int count1 = orders1.size();
        System.out.println("First count: " + count1);  // e.g., 50 orders
        
        // Calculate statistics
        BigDecimal totalAmount1 = orders1.stream()
            .map(Order::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        System.out.println("First total: $" + totalAmount1);  // e.g., $25,000
        
        // Simulate processing time
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // Meanwhile, another transaction inserts new orders
        // INSERT INTO orders (status, date, amount) VALUES ('PENDING', '2024-01-15', 1000);
        
        // Second query - SAME query, SAME transaction
        List<Order> orders2 = orderRepo.findByStatusAndDate(status, date);
        int count2 = orders2.size();
        System.out.println("Second count: " + count2);  // e.g., 52 orders
        
        BigDecimal totalAmount2 = orders2.stream()
            .map(Order::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        System.out.println("Second total: $" + totalAmount2);  // e.g., $27,000
        
        // New rows appeared!
        if (count1 != count2) {
            System.out.println("PHANTOM READ DETECTED! ❌");
            System.out.println("First: " + count1 + " orders, Second: " + count2 + " orders");
            System.out.println("Phantom rows: " + (count2 - count1));
        }
    }
}
```

---

### Types of Phantom Reads

#### 1. Phantom Insert (Most Common)

```java
// Query 1: SELECT * FROM orders WHERE status = 'PENDING';
// Returns: 10 rows

// Another transaction: INSERT INTO orders (status) VALUES ('PENDING');

// Query 2: SELECT * FROM orders WHERE status = 'PENDING';
// Returns: 11 rows ← PHANTOM INSERT
```

#### 2. Phantom Delete

```java
// Query 1: SELECT * FROM orders WHERE status = 'PENDING';
// Returns: 10 rows

// Another transaction: DELETE FROM orders WHERE id = 5 AND status = 'PENDING';

// Query 2: SELECT * FROM orders WHERE status = 'PENDING';
// Returns: 9 rows ← PHANTOM DELETE
```

#### 3. Phantom Update (Row enters/leaves range)

```java
// Query 1: SELECT * FROM account WHERE balance > 1000;
// Returns: Account 5 (balance = 1500)

// Another transaction: UPDATE account SET balance = 500 WHERE id = 5;

// Query 2: SELECT * FROM account WHERE balance > 1000;
// Returns: Account 5 is GONE ← PHANTOM (left the range)
```

---

### How to Prevent Phantom Reads

Use **SERIALIZABLE** isolation level:

```java
// ✅ SOLUTION: Use SERIALIZABLE
@Transactional(isolation = Isolation.SERIALIZABLE)
public StatisticalReport generateConsistentStats(BigDecimal threshold) {
    
    // First query
    List<Account> accounts1 = accountRepo.findByBalanceGreaterThan(threshold);
    int count1 = accounts1.size();  // e.g., 100
    
    // Database places RANGE LOCK
    // Other transactions CANNOT insert/delete accounts matching this range
    
    // Processing...
    
    // Second query - GUARANTEED same number of rows
    List<Account> accounts2 = accountRepo.findByBalanceGreaterThan(threshold);
    int count2 = accounts2.size();  // Still 100 ✅
    
    // count1 == count2 (GUARANTEED)
    
    return generateReport(accounts1);
}
```

---

### Special Case: MySQL InnoDB

**Note:** MySQL InnoDB with **REPEATABLE_READ** actually **prevents phantom reads** using "next-key locking" (combination of row locks and gap locks). This is **non-standard** behavior.

```java
// In MySQL InnoDB ONLY:
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void mysqlSpecialCase() {
    // MySQL prevents phantom reads at REPEATABLE_READ level
    // Most other databases require SERIALIZABLE
}
```

---

### Visual Timeline

```
Time    Transaction 1                          Transaction 2
--------------------------------------------------------------------
t1      START TRANSACTION (REPEATABLE_READ)              
t2      SELECT * WHERE balance > 1000          
        (returns 5 rows)
t3                                             START TRANSACTION
t4                                             INSERT balance = 5000
t5                                             COMMIT ✅
t6      SELECT * WHERE balance > 1000          
        (returns 6 rows) ← PHANTOM READ!
t7      COMMIT                             

------- WITH SERIALIZABLE -------

Time    Transaction 1                          Transaction 2
--------------------------------------------------------------------
t1      START TRANSACTION (SERIALIZABLE)                 
t2      SELECT * WHERE balance > 1000          
        [Holds RANGE LOCK]
t3                                             START TRANSACTION
t4                                             INSERT balance = 5000
t5                                             ⏳ BLOCKED by range lock
t6      SELECT * WHERE balance > 1000          
        (returns 5 rows again) ✅
t7      COMMIT [releases range lock]           
t8                                             ✅ Now can insert
```

---

## 4. Lost Update

### What is a Lost Update?

A **lost update** occurs when two transactions read the same data, both modify it based on the original value, and the second transaction's update **overwrites** the first transaction's changes - causing the first update to be **lost**.

### Why is it called "Lost"?

Because one transaction's update is **completely lost** and overwritten by another transaction, as if it never happened.

---

### Example Scenario: Bank Account Withdrawal

```java
// Initial state: Account balance = $500

// ============================================
// Transaction 1: Wife withdraws $100
// ============================================
START TRANSACTION;
SELECT balance FROM account WHERE id = 1;  
// Reads: $500

// Calculate new balance
newBalance = 500 - 100;  // = $400

// ============================================
// Transaction 2: Husband withdraws $200
// ============================================
START TRANSACTION;
SELECT balance FROM account WHERE id = 1;  
// Reads: $500 (same original value)

// Calculate new balance
newBalance = 500 - 200;  // = $300

// ============================================
// Transaction 1: Commits first
// ============================================
UPDATE account SET balance = 400 WHERE id = 1;
COMMIT;  // Balance is now $400

// ============================================
// Transaction 2: Commits second (OVERWRITES T1!)
// ============================================
UPDATE account SET balance = 300 WHERE id = 1;
COMMIT;  // Balance is now $300 (WRONG!)

// EXPECTED: $500 - $100 - $200 = $200
// ACTUAL: $300
// LOST: Wife's $100 withdrawal ❌
```

**The Problem:**
- Both transactions read balance = $500
- Wife withdrew $100 (balance should be $400)
- Husband withdrew $200 (balance should be $200)
- Final balance is $300 instead of $200
- Wife's withdrawal was **completely lost**

---

### Real-World Impact

```java
// E-commerce Inventory Example
@Transactional(isolation = Isolation.READ_COMMITTED)
public void processOrder(Long productId, int quantity) {
    
    // Customer 1's transaction
    Product product1 = productRepo.findById(productId).orElseThrow();
    int stock1 = product1.getStock();  // Read: 10 items
    
    // Reduce stock
    product1.setStock(stock1 - 3);  // 10 - 3 = 7
    
    // Meanwhile, Customer 2's transaction runs
    Product product2 = productRepo.findById(productId).orElseThrow();
    int stock2 = product2.getStock();  // Read: 10 items (same original value!)
    product2.setStock(stock2 - 5);  // 10 - 5 = 5
    
    // Customer 1 saves
    productRepo.save(product1);  // Stock = 7
    
    // Customer 2 saves (OVERWRITES Customer 1!)
    productRepo.save(product2);  // Stock = 5 (WRONG!)
    
    // EXPECTED: 10 - 3 - 5 = 2 items left
    // ACTUAL: 5 items left
    // LOST: Customer 1's purchase of 3 items ❌
    // PROBLEM: Inventory is wrong, might oversell!
}
```

---

### Code Example with Problem

```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepo;
    
    // ❌ PROBLEM: Lost update without locking
    @Transactional
    public void updateStockWithoutLock(Long productId, int quantitySold) {
        
        // Read current stock
        Product product = productRepo.findById(productId).orElseThrow();
        int currentStock = product.getStock();  // e.g., 100
        
        // Another transaction might read the same value here
        
        // Calculate new stock
        int newStock = currentStock - quantitySold;
        
        // Another transaction might also calculate based on original value
        
        // Update stock
        product.setStock(newStock);
        productRepo.save(product);
        
        // If another transaction saves after this,
        // it will OVERWRITE this update!
        // LOST UPDATE ❌
    }
}
```

---

### Solutions to Prevent Lost Updates

#### Solution 1: Pessimistic Write Lock (SELECT FOR UPDATE)

```java
// ✅ SOLUTION 1: Use pessimistic write lock
@Transactional
public void updateStockWithPessimisticLock(Long productId, int quantitySold) {
    
    // Acquire exclusive lock immediately
    Product product = productRepo.findByIdWithWriteLock(productId).orElseThrow();
    
    // Other transactions CANNOT read this row until we commit
    
    int currentStock = product.getStock();
    product.setStock(currentStock - quantitySold);
    productRepo.save(product);
    
    // Lock released on commit
    // Now other transactions can proceed
}

// Repository method
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithWriteLock(@Param("id") Long id);
}
```

#### Solution 2: Optimistic Locking with @Version

```java
// ✅ SOLUTION 2: Use optimistic locking
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    private Long id;
    
    @Version  // Optimistic locking
    private Long version;
    
    private Integer stock;
    
    // Getters and setters
}

@Service
public class ProductService {
    
    @Transactional
    public void updateStockWithOptimisticLock(Long productId, int quantitySold) {
        try {
            Product product = productRepo.findById(productId).orElseThrow();
            
            // Read: stock = 100, version = 1
            
            int currentStock = product.getStock();
            product.setStock(currentStock - quantitySold);
            
            // Save generates SQL:
            // UPDATE products 
            // SET stock = ?, version = 2 
            // WHERE id = ? AND version = 1
            
            productRepo.save(product);
            // If version changed (another transaction updated),
            // OptimisticLockException is thrown
            
        } catch (OptimisticLockException e) {
            // Retry the operation
            throw new ConcurrentModificationException("Product was updated by another transaction");
        }
    }
}
```

#### Solution 3: Atomic Database Operations

```java
// ✅ SOLUTION 3: Use atomic update at database level
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    @Modifying
    @Query("UPDATE Product p SET p.stock = p.stock - :quantity WHERE p.id = :id")
    int decrementStock(@Param("id") Long id, @Param("quantity") int quantity);
}

@Service
public class ProductService {
    
    @Transactional
    public void updateStockAtomically(Long productId, int quantitySold) {
        // Single atomic operation
        // UPDATE products SET stock = stock - ? WHERE id = ?
        int updated = productRepo.decrementStock(productId, quantitySold);
        
        if (updated == 0) {
            throw new ProductNotFoundException(productId);
        }
        
        // No lost update possible - database handles it atomically
    }
}
```

---

### Comparison of Solutions

| Solution | Pros | Cons | Best For |
|----------|------|------|----------|
| **Pessimistic Lock** | - Guarantees no conflicts<br>- Simple to understand | - Can cause contention<br>- Slower performance<br>- Risk of deadlocks | High contention, critical operations |
| **Optimistic Lock** | - Better performance<br>- No deadlocks<br>- Works well with low contention | - Requires retry logic<br>- Transaction may fail | Low contention, read-heavy workloads |
| **Atomic Operations** | - Best performance<br>- No locking overhead<br>- No lost updates | - Limited to simple operations<br>- Can't do complex calculations | Simple increment/decrement operations |

---

### Visual Timeline

```
WITHOUT LOCKING (Lost Update):

Time    Transaction 1              Transaction 2              Database
------------------------------------------------------------------------
t1      READ balance = $500                                   $500
t2                                 READ balance = $500        $500
t3      balance = 500 - 100                                   $500
t4                                 balance = 500 - 200        $500
t5      UPDATE balance = $400                                 $400 ✓
t6                                 UPDATE balance = $300      $300 ✓
t7      ❌ Lost $100 withdrawal!                              $300 (WRONG!)

Expected: $500 - $100 - $200 = $200
Actual: $300


WITH PESSIMISTIC LOCKING (No Lost Update):

Time    Transaction 1                  Transaction 2          Database
------------------------------------------------------------------------
t1      SELECT FOR UPDATE              
        [Acquires exclusive lock]
        READ balance = $500                                   $500
t2                                     SELECT FOR UPDATE      
                                       ⏳ BLOCKED (waiting)
t3      balance = 500 - 100                                   $500
t4      UPDATE balance = $400                                 $400
t5      COMMIT [releases lock]                                $400 ✓
t6                                     ✅ Lock acquired
                                       READ balance = $400    $400
t7                                     balance = 400 - 200    $400
t8                                     UPDATE balance = $200  $200 ✓
t9                                     COMMIT                 $200

Expected: $200
Actual: $200 ✅
```

---

## Comparison Table

| Problem | What Happens | Example | Prevented By |
|---------|-------------|---------|--------------|
| **Dirty Read** | Reading **uncommitted data** from another transaction | Read balance = $1000, but transaction rolls back (was actually $500) | READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE |
| **Non-Repeatable Read** | **Same row** returns **different values** on repeated reads | Read balance = $500, read again = $1000 (value changed) | REPEATABLE_READ, SERIALIZABLE |
| **Phantom Read** | **Different number of rows** returned on repeated queries | Query returns 5 rows, query again returns 6 rows (new row inserted) | SERIALIZABLE |
| **Lost Update** | One transaction's update **overwrites** another's | Both
