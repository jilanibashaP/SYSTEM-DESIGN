# Database Concurrency Control
## What Are Concurrency Problems?

**Definition**: When multiple transactions run simultaneously, they can interfere with each other and cause data inconsistency.

**Why do these problems happen?**
- Multiple users accessing the same data at the same time
- One transaction reads data while another is modifying it
- Lack of proper isolation between transactions

**Solution**: Database isolation levels and locking mechanisms

---

## 1. Dirty Read 🔴

### Definition
A **dirty read** occurs when a transaction reads data that has been **modified but not yet committed** by another transaction. If the modifying transaction rolls back, the data read was never valid.

### Why Called "Dirty"?
Reading **uncommitted, potentially invalid data** - data that might be "rolled back" and never actually exist in the database.

---

### SQL Example: Bank Account Transfer

```sql
-- Initial state: Account balance = $500

-- ============================================
-- Transaction 1: Attempting to transfer money
-- ============================================
START TRANSACTION;
UPDATE account SET balance = 1000 WHERE id = 1;  
-- Balance is now $1000 (but NOT committed yet!)

-- ============================================
-- Transaction 2: Checking balance
-- ============================================
START TRANSACTION;
SELECT balance FROM account WHERE id = 1;  
-- Reads $1000 (DIRTY READ - uncommitted data!)

-- Transaction 2 makes a decision based on $1000 balance
-- if (balance >= 900) then processLoan();  
-- Loan approved based on dirty data!
COMMIT;

-- ============================================
-- Transaction 1: Something goes wrong
-- ============================================
ROLLBACK;  
-- Balance returns to $500 (the update never happened!)
```

**The Problem:**
- Transaction 2 saw balance = $1000
- Transaction 1 rolled back, so balance was actually $500 all along
- Transaction 2 approved a loan based on **data that never existed**

---

### Real-World Impact: E-commerce Example

```sql
-- Transaction 1: User adds item to cart
START TRANSACTION;
UPDATE inventory SET quantity = 5 WHERE product_id = 101;
-- Reduced from 10 to 5 (not committed)

-- Transaction 2: Admin checks inventory
SELECT quantity FROM inventory WHERE product_id = 101;
-- Sees 5 items (DIRTY READ)
-- Sends alert: "Low stock! Only 5 left"

-- Transaction 1: User cancels
ROLLBACK;
-- Quantity is actually still 10!
```

**Impact**: False low-stock alerts, incorrect business decisions based on uncommitted data

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

### How to Prevent
Use **READ_COMMITTED** isolation level or higher - ensures you only read committed data.

---

## 2. Non-Repeatable Read 🟡

### Definition
A **non-repeatable read** occurs when a transaction reads the **same row twice** and gets **different values** because another transaction modified and committed the data between the two reads.

### Why Called "Non-Repeatable"?
If you repeat the same read query within your transaction, you get **different results** - the read cannot be repeated with consistent results.

---

### SQL Example: Monthly Report Generation

```sql
-- Initial state: Account balance = $500

-- ============================================
-- Transaction 1: Generating monthly report
-- ============================================
START TRANSACTION;

-- First read at beginning of report
SELECT balance FROM account WHERE id = 1;  
-- Returns $500

-- ... processing other data ...

-- ============================================
-- Transaction 2: Customer deposits money
-- ============================================
START TRANSACTION;
UPDATE account SET balance = 1000 WHERE id = 1;
COMMIT;  -- ✅ Committed!

-- ============================================
-- Transaction 1: Reading again for verification
-- ============================================
-- Second read at end of report
SELECT balance FROM account WHERE id = 1;  
-- Returns $1000 (NON-REPEATABLE READ - different value!)

COMMIT;
```

**The Problem:**
- Transaction 1 read balance as $500
- Transaction 1 read balance again as $1000
- Same query, same transaction, **different results**
- Report has **inconsistent data** ($500 in one place, $1000 in another)

---

### Real-World Impact: Financial Report

```sql
-- First read: Get opening balance
SELECT balance FROM account WHERE id = 1;
-- Returns $500

-- Processing transactions for the month...
-- Calculating debits, credits, etc.

-- Meanwhile, another transaction updates the account:
-- UPDATE account SET balance = 1000 WHERE id = 1;
-- COMMIT;

-- Second read: Get closing balance
SELECT balance FROM account WHERE id = 1;
-- Returns $1000 (CHANGED!)

-- Calculate expected vs actual:
-- Expected balance = $800 (based on opening + transactions)
-- Actual balance = $1000 
-- MISMATCH! Report is inconsistent ❌
```

**Impact**: Financial reports with inconsistent numbers, failed audits, incorrect business decisions

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

### How to Prevent
Use **REPEATABLE_READ** or **SERIALIZABLE** isolation level - ensures the same query returns the same results throughout the transaction.

---

## 3. Phantom Read 👻

### Definition
A **phantom read** occurs when a transaction executes the **same query twice** and gets a **different number of rows** because another transaction **inserted or deleted rows** that match the query criteria.

### Why Called "Phantom"?
Because **new rows appear** (or existing rows disappear) like "phantoms" between two identical queries in the same transaction.

---

### Key Difference from Non-Repeatable Read

| Problem | What Changes | Example |
|---------|-------------|---------|
| **Non-Repeatable Read** | **Existing row values** change | Row 1: balance changes from $500 to $1000 |
| **Phantom Read** | **Number of rows** changes | Query returns 5 rows, then returns 6 rows |

---

### SQL Example: Account Summary Report

```sql
-- Initial state: 5 accounts with balance > $1000

-- ============================================
-- Transaction 1: Generating high-balance report
-- ============================================
START TRANSACTION;

-- First query: Find all high-balance accounts
SELECT * FROM account WHERE balance > 1000;  
-- Returns 5 rows:
-- Account 1: $1500
-- Account 2: $2000
-- Account 3: $1200
-- Account 4: $3000
-- Account 5: $1800

-- Count: 5 accounts
-- Total: $9500

-- ... processing data ...

-- ============================================
-- Transaction 2: New customer opens account
-- ============================================
START TRANSACTION;
INSERT INTO account (id, balance) VALUES (6, 5000);
COMMIT;  -- ✅ New account added!

-- ============================================
-- Transaction 1: Re-running query for verification
-- ============================================
-- Second query: Same query, same transaction
SELECT * FROM account WHERE balance > 1000;  
-- Returns 6 rows (PHANTOM READ - new row appeared!):
-- Account 1: $1500
-- Account 2: $2000
-- Account 3: $1200
-- Account 4: $3000
-- Account 5: $1800
-- Account 6: $5000  ← PHANTOM ROW!

-- Count: 6 accounts (count changed!)
-- Total: $14500 (total changed!)

COMMIT;
```

**The Problem:**
- Transaction 1 counted 5 accounts with total $9500
- Transaction 1 counted again: 6 accounts with total $14500
- Same query, **different number of rows**
- Report has **inconsistent counts and totals**

---

### Real-World Impact: Statistical Report

```sql
-- First query: Count high-balance accounts
SELECT COUNT(*), AVG(balance) FROM account WHERE balance > 1000;
-- Returns: 100 accounts, average $5000

-- Processing and analyzing demographics based on 100 accounts...

-- Meanwhile, other transactions modify data:
-- INSERT INTO account (balance) VALUES (10000);
-- INSERT INTO account (balance) VALUES (15000);
-- DELETE FROM account WHERE balance = 5500;
-- COMMIT;

-- Second query: Re-count for final report
SELECT COUNT(*), AVG(balance) FROM account WHERE balance > 1000;
-- Returns: 102 accounts, average $5200 (CHANGED!)

-- INCONSISTENCY:
-- Demographics analysis based on 100 accounts
-- Final report shows 102 accounts
-- Average balance different
-- Report is INVALID! ❌
```

**Impact**: Statistical reports with inconsistent data, wrong business insights, failed compliance checks

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

### How to Prevent
Use **SERIALIZABLE** isolation level - uses range locks to prevent new rows from being inserted or deleted within your query range.

---

## Comparison Table

| Problem | What Happens | Example | Prevented By |
|---------|-------------|---------|--------------|
| **Dirty Read** | Reading **uncommitted data** from another transaction | Read balance = $1000, but transaction rolls back (was actually $500) | READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE |
| **Non-Repeatable Read** | **Same row** returns **different values** on repeated reads | Read balance = $500, read again = $1000 (value changed) | REPEATABLE_READ, SERIALIZABLE |
| **Phantom Read** | **Different number of rows** returned on repeated queries | Query returns 5 rows, query again returns 6 rows (new row inserted) | SERIALIZABLE |

---

## Isolation Levels

### Complete Isolation Level Matrix

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance | Consistency |
|-----------------|------------|---------------------|--------------|-------------|-------------|
| **READ UNCOMMITTED** (Level 0) | ✗ Possible | ✗ Possible | ✗ Possible | ⚡⚡⚡⚡ | ⬇️ Lowest |
| **READ COMMITTED** (Level 1) | ✅ Prevented | ✗ Possible | ✗ Possible | ⚡⚡⚡ | ⬆️ Medium |
| **REPEATABLE READ** (Level 2) | ✅ Prevented | ✅ Prevented | ✗ Possible | ⚡⚡ | ⬆️ High |
| **SERIALIZABLE** (Level 3) | ✅ Prevented | ✅ Prevented | ✅ Prevented | ⚡ | ⬆️ Highest |

**Key Points:**
- ✅ **Prevented** = This isolation level guarantees this problem won't occur
- ✗ **Possible** = This isolation level allows this problem to happen
- **Higher isolation level = More consistency, but Lower performance**

---

## Locking Strategies by Isolation Level

### Level 0: READ UNCOMMITTED

```
┌─────────────────────────────────────────┐
│ READ UNCOMMITTED                         │
├─────────────────────────────────────────┤
│ Read:  No Lock acquired                 │
│ Write: No Lock acquired                 │
└─────────────────────────────────────────┘
```

**Behavior:**
- No locks at all - can read uncommitted data
- Transactions can read data that other transactions are modifying
- Can see "dirty" data that gets rolled back

**Example:**
```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public BigDecimal getBalance(Long accountId) {
    // No locks acquired - fastest but least safe
    Account account = accountRepo.findById(accountId).orElseThrow();
    return account.getBalance();  // Might read uncommitted changes
}
```

---

### Level 1: READ COMMITTED

```
┌─────────────────────────────────────────────────────────────┐
│ READ COMMITTED                                               │
├─────────────────────────────────────────────────────────────┤
│ Read:  Shared Lock acquired and Released immediately        │
│        after read is done                                   │
│ Write: Exclusive Lock acquired and kept until transaction   │
│        commits                                              │
└─────────────────────────────────────────────────────────────┘
```

**Explanation:**
- **Read Operations:** 
  - Acquires **Shared Lock** (S-Lock)
  - Reads the data
  - **Releases lock immediately** after read completes
  - Multiple transactions can read simultaneously
  
- **Write Operations:**
  - Acquires **Exclusive Lock** (X-Lock)
  - **Holds lock until transaction commits**
  - Blocks all other transactions from reading or writing

**Timeline Example:**
```
Time    Transaction 1                          Transaction 2
--------------------------------------------------------------------
t1      SELECT balance (acquire S-Lock)        
t2      Read: $500                             
t3      Release S-Lock immediately             
t4                                             UPDATE balance = $1000
t5                                             (acquire X-Lock, hold it)
t6      SELECT balance (acquire S-Lock)        
t7      Read: $500 (T2 not committed yet)      
t8      Release S-Lock                         
t9                                             COMMIT (release X-Lock)
t10     SELECT balance                         
t11     Read: $1000 (now sees T2's changes)    
```

**Why Non-Repeatable Reads Happen:**
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void demonstrateReadCommitted(Long accountId) {
    
    // Read 1: Acquire S-Lock, read, release immediately
    Account account1 = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance1 = account1.getBalance();  // $500
    // S-Lock RELEASED here!
    
    // Another transaction can now update this row
    // UPDATE account SET balance = 1000 WHERE id = accountId;
    // COMMIT;
    
    // Read 2: Acquire new S-Lock, read, release
    Account account2 = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance2 = account2.getBalance();  // $1000 (CHANGED!)
    // S-Lock RELEASED here!
    
    // balance1 != balance2 (Non-Repeatable Read)
}
```

---

### Level 2: REPEATABLE READ

```
┌─────────────────────────────────────────────────────────────┐
│ REPEATABLE READ                                              │
├─────────────────────────────────────────────────────────────┤
│ Read:  Shared Lock acquired and Released ONLY at the end    │
│        of the Transaction                                   │
│ Write: Exclusive Lock acquired and Released ONLY at the     │
│        end of the Transaction                               │
└─────────────────────────────────────────────────────────────┘
```

**Explanation:**
- **Read Operations:**
  - Acquires **Shared Lock** (S-Lock)
  - **Holds lock until transaction ends** (commit/rollback)
  - Same row always returns same value within transaction
  
- **Write Operations:**
  - Acquires **Exclusive Lock** (X-Lock)
  - **Holds lock until transaction ends**
  - Completely blocks other transactions

**Timeline Example:**
```
Time    Transaction 1                          Transaction 2
--------------------------------------------------------------------
t1      SELECT balance (acquire S-Lock)        
t2      Read: $500                             
t3      (HOLDING S-Lock)                       
t4                                             UPDATE balance = $1000
t5                                             ⏳ BLOCKED (waiting for S-Lock)
t6      SELECT balance (still holding S-Lock)  
t7      Read: $500 (SAME value guaranteed)     
t8      COMMIT (release S-Lock)                
t9                                             ✅ Now can acquire X-Lock
t10                                            UPDATE succeeds
t11                                            COMMIT
```

**Why This Prevents Non-Repeatable Reads:**
```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void demonstrateRepeatableRead(Long accountId) {
    
    // Read 1: Acquire S-Lock, read, KEEP HOLDING
    Account account1 = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance1 = account1.getBalance();  // $500
    // S-Lock STILL HELD!
    
    // Another transaction tries to update
    // UPDATE account SET balance = 1000 WHERE id = accountId;
    // ⏳ BLOCKED - must wait for our S-Lock to be released
    
    // Read 2: Re-read (still holding S-Lock)
    Account account2 = accountRepo.findById(accountId).orElseThrow();
    BigDecimal balance2 = account2.getBalance();  // $500 (SAME!)
    // S-Lock STILL HELD!
    
    // balance1 == balance2 (Repeatable Read guaranteed)
    
    // COMMIT - S-Lock finally released
}
```

**Why Phantom Reads Still Happen:**
- Row locks prevent existing rows from changing
- But **don't prevent new rows from being inserted**
- Range queries can return different number of rows

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void phantomReadExample() {
    
    // Query 1: Lock all rows that match
    List<Account> accounts1 = accountRepo.findByBalanceGreaterThan(1000);
    // Locks: Row 1, Row 2, Row 3, Row 4, Row 5
    int count1 = accounts1.size();  // 5 rows
    
    // Another transaction can INSERT a new row (not locked yet!)
    // INSERT INTO account (balance) VALUES (5000);  ✅ Allowed!
    // COMMIT;
    
    // Query 2: Same query
    List<Account> accounts2 = accountRepo.findByBalanceGreaterThan(1000);
    // Now sees: Row 1, Row 2, Row 3, Row 4, Row 5, Row 6
    int count2 = accounts2.size();  // 6 rows (PHANTOM!)
}
```

---

### Level 3: SERIALIZABLE

```
┌─────────────────────────────────────────────────────────────┐
│ SERIALIZABLE                                                 │
├─────────────────────────────────────────────────────────────┤
│ Same as Repeatable Read Locking Strategy                    │
│ + Apply RANGE LOCK                                          │
│ + All locks Released ONLY at the end of Transaction         │
└─────────────────────────────────────────────────────────────┘
```

**Explanation:**
- **Everything from REPEATABLE READ, PLUS:**
  - **Range Locks** on query predicates
  - Prevents INSERT, UPDATE, DELETE in the locked range
  - Complete isolation - transactions execute as if they're running one after another

**How Range Locks Work:**
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void serializableExample() {
    
    // Query: Find accounts with balance > 1000
    List<Account> accounts = accountRepo.findByBalanceGreaterThan(1000);
    
    // RANGE LOCK applied on: "balance > 1000"
    // Locks:
    // - All existing rows matching condition
    // - The "gap" where new matching rows could be inserted
    
    // Other transactions are now BLOCKED from:
    // ❌ INSERT INTO account (balance) VALUES (5000);  -- Would match range
    // ❌ UPDATE account SET balance = 2000 WHERE id = 10;  -- Would enter range
    // ❌ DELETE FROM account WHERE balance = 1500;  -- Would affect range
    
    // Range lock prevents ALL modifications to the result set
    // Guarantees: Same query returns exact same rows
}
```

**Timeline Example:**
```
Time    Transaction 1                          Transaction 2
--------------------------------------------------------------------
t1      START TRANSACTION (SERIALIZABLE)                 
t2      SELECT * WHERE balance > 1000          
        [Acquires RANGE LOCK on "balance > 1000"]
        Returns: 5 rows
t3                                             START TRANSACTION
t4                                             INSERT (balance=5000)
t5                                             ⏳ BLOCKED by range lock
t6                                             UPDATE SET balance=2000
t7                                             ⏳ BLOCKED by range lock
t8      SELECT * WHERE balance > 1000          
        Returns: 5 rows (SAME! No phantoms) ✅
t9      COMMIT [releases range lock]           
t10                                            ✅ Now can proceed
t11                                            INSERT succeeds
t12                                            COMMIT
```

---

## Distributed Concurrency Control

In distributed systems (microservices, distributed databases), we need different strategies for managing concurrency because traditional database locks don't work across multiple services or databases.

### Two Main Approaches

```
                    Using Distributed Concurrency Control
                                    |
                    +---------------+----------------+
                    |                                |
        Optimistic Concurrency Control (OCC)    Pessimistic Concurrency Control (PCC)
```

---

## Optimistic Concurrency Control (OCC)

### Characteristics

**Philosophy:** "Assume conflicts are rare, detect and handle them when they occur"

**Key Features:**
- **Isolation Level used:** Below REPEATABLE READ (typically READ_COMMITTED)
- **Concurrency:** Much Higher - transactions don't block each other
- **Deadlock:** No chance of Deadlock - no locks acquired
- **Conflict Handling:** In case of conflict, overhead of transaction rollback and retry logic is there

---

### How OCC Works

```
┌─────────────────────────────────────────────────────────────┐
│ OPTIMISTIC CONCURRENCY CONTROL (OCC)                         │
├─────────────────────────────────────────────────────────────┤
│ 1. READ Phase:    Read data without locks                   │
│ 2. MODIFY Phase:  Make changes in memory/local cache        │
│ 3. VALIDATE Phase: Check if data changed since read         │
│ 4. WRITE Phase:   If valid, commit; else rollback & retry   │
└─────────────────────────────────────────────────────────────┘
```

**Example with @Version:**
```java
@Entity
public class Order {
    @Id
    private Long id;
    
    @Version  // Optimistic lock
    private Long version;
    
    private String status;
    private BigDecimal amount;
}

@Service
public class OrderService {
    
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void updateOrderWithRetry(Long orderId, String newStatus) {
        int maxRetries = 3;
        int attempt = 0;
        
        while (attempt < maxRetries) {
            try {
                // 1. READ: Get current state (version = 5)
                Order order = orderRepo.findById(orderId).orElseThrow();
                
                // 2. MODIFY: Change in memory
                order.setStatus(newStatus);
                
                // 3. VALIDATE & 4. WRITE:
                // UPDATE orders SET status=?, version=6 WHERE id=? AND version=5
                orderRepo.save(order);
                
                // Success!
                return;
                
            } catch (OptimisticLockException e) {
                // Another transaction modified this order
                attempt++;
                if (attempt >= maxRetries) {
                    throw new ConcurrentModificationException(
                        "Order was modified by another transaction after " + maxRetries + " retries"
                    );
                }
                // Retry after brief delay
                Thread.sleep(100 * attempt);
            }
        }
    }
}
```

**Timeline:**
```
Time    Transaction 1 (T1)             Transaction 2 (T2)         Database
-------------------------------------------------------------------------------
t1      READ order (version=5)                                    version=5
t2                                     READ order (version=5)     version=5
t3      MODIFY status="SHIPPED"                                   version=5
t4                                     MODIFY status="CANCELLED"  version=5
t5      WRITE (version=5→6)                                       version=6 ✓
t6                                     WRITE (version=5→6)        
t7                                     ❌ FAIL! version mismatch
t8                                     ROLLBACK                   version=6
t9                                     RETRY: READ (version=6)    version=6
t10                                    MODIFY status="CANCELLED"  version=6
t11                                    WRITE (version=6→7)        version=7 ✓
```

---

### OCC in Distributed Systems

**Scenario:** Microservices updating shared state

```java
// Order Service
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Transactional
    public void processOrder(OrderRequest request) {
        
        // 1. READ: Get inventory from Inventory Service
        InventoryDTO inventory = restTemplate.getForObject(
            "http://inventory-service/api/inventory/" + request.getProductId(),
            InventoryDTO.class
        );
        
        Long currentVersion = inventory.getVersion();  // version = 10
        Integer currentStock = inventory.getStock();    // stock = 100
        
        // 2. MODIFY: Calculate new stock
        Integer newStock = currentStock - request.getQuantity();
        
        // 3. VALIDATE & 4. WRITE: Update with version check
        UpdateRequest updateRequest = new UpdateRequest();
        updateRequest.setStock(newStock);
        updateRequest.setExpectedVersion(currentVersion);  // Must be 10
        
        try {
            restTemplate.put(
                "http://inventory-service/api/inventory/" + request.getProductId(),
                updateRequest
            );
            // Success!
        } catch (OptimisticLockException e) {
            // Version mismatch - another service updated inventory
            throw new ConcurrentUpdateException("Inventory changed, please retry");
        }
    }
}

// Inventory Service
@RestController
public class InventoryController {
    
    @PutMapping("/api/inventory/{id}")
    public ResponseEntity<?> updateInventory(
            @PathVariable Long id,
            @RequestBody UpdateRequest request) {
        
        // Update only if version matches
        int updated = inventoryRepo.updateWithVersionCheck(
            id, 
            request.getStock(), 
            request.getExpectedVersion()
        );
        
        if (updated == 0) {
            // Version mismatch
            throw new OptimisticLockException("Inventory version changed");
        }
        
        return ResponseEntity.ok().build();
    }
}
```

---

### When to Use OCC

**✅ Best For:**
- Low contention scenarios (conflicts are rare)
- Read-heavy workloads
- Distributed systems where locking is expensive
- Systems requiring high throughput
- Short transactions

**❌ Avoid When:**
- High contention (many concurrent updates)
- Long-running transactions
- Cannot afford retry overhead
- Conflicts are expensive to resolve

---

## Pessimistic Concurrency Control (PCC)

### Characteristics

**Philosophy:** "Assume conflicts will happen, prevent them with locks"

**Key Features:**
- **Isolation Level used:** REPEATABLE READ and SERIALIZABLE
- **Concurrency:** Less Concurrency compared to Optimistic
- **Deadlock:** Deadlock is Possible, then transactions stuck in deadlock forced to do rollback
- **Locking:** Putting a long lock, sometimes timeout issue comes and need to be done

---

### How PCC Works

```
┌─────────────────────────────────────────────────────────────┐
│ PESSIMISTIC CONCURRENCY CONTROL (PCC)                        │
├─────────────────────────────────────────────────────────────┤
│ 1. LOCK Phase:   Acquire lock BEFORE reading                │
│ 2. READ Phase:   Read data (lock held)                      │
│ 3. MODIFY Phase: Make changes (lock held)                   │
│ 4. WRITE Phase:  Commit changes                             │
│ 5. UNLOCK Phase: Release lock                               │
└─────────────────────────────────────────────────────────────┘
```

**Example with SELECT FOR UPDATE:**
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT o FROM Order o WHERE o.id = :id")
    Optional<Order> findByIdWithLock(@Param("id") Long id);
}

@Service
public class OrderService {
    
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void updateOrderWithLock(Long orderId, String newStatus) {
        
        // 1. LOCK: Acquire exclusive lock immediately
        // SQL: SELECT * FROM orders WHERE id = ? FOR UPDATE
        Order order = orderRepo.findByIdWithLock(orderId)
            .orElseThrow();
        
        // Other transactions are BLOCKED here until we commit
        
        // 2. READ: Data is locked
        String currentStatus = order.getStatus();
        
        // 3. MODIFY: Make changes
        order.setStatus(newStatus);
        
        // 4. WRITE: Save changes
        orderRepo.save(order);
        
        // 5. UNLOCK: Lock released on commit
    }
}
```

**Timeline:**
```
Time    Transaction 1 (T1)                 Transaction 2 (T2)         Database
-----------------------------------------------------------------------------------
t1      SELECT FOR UPDATE                                            
        [Acquires WRITE LOCK]
        READ order                                                   LOCKED by T1
t2                                         SELECT FOR UPDATE         
t3                                         ⏳ BLOCKED (waiting)      LOCKED by T1
t4      MODIFY status="SHIPPED"                                      LOCKED by T1
t5      WRITE changes                                                LOCKED by T1
t6      COMMIT [releases lock]                                       UNLOCKED ✓
t7                                         ✅ Lock acquired          LOCKED by T2
t8                                         READ order                LOCKED by T2
t9                                         MODIFY status="DELIVERED" LOCKED by T2
t10                                        WRITE changes             LOCKED by T2
t11                                        COMMIT [releases lock]    UNLOCKED ✓
```

---

### Deadlock Problem with PCC

**What is Deadlock?**
Two or more transactions waiting for each other to release locks, creating a circular dependency.

```
┌──────────────────────────────────────────────────────────┐
│                    DEADLOCK SCENARIO                      │
├──────────────────────────────────────────────────────────┤
│ T1: Locks Resource A, waits for Resource B               │
│ T2: Locks Resource B, waits for Resource A               │
│                                                           │
│ Result: Both transactions stuck forever! ⚠️              │
└──────────────────────────────────────────────────────────┘
```

**Example:**
```java
// Transaction 1
@Transactional
public void transferMoney(Long fromAccountId, Long toAccountId, BigDecimal amount) {
    
    // T1: Lock Account A
    Account fromAccount = accountRepo.findByIdWithLock(fromAccountId).orElseThrow();
    
    // T1: Now trying to lock Account B (but T2 already has it!)
    Account toAccount = accountRepo.findByIdWithLock(toAccountId).orElseThrow();
    
    fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
    toAccount.setBalance(toAccount.getBalance().add(amount));
    
    accountRepo.save(fromAccount);
    accountRepo.save(toAccount);
}

// Transaction 2 (running simultaneously)
@Transactional
public void transferMoney(Long fromAccountId, Long toAccountId, BigDecimal amount) {
    
    // T2: Lock Account B
    Account fromAccount = accountRepo.findByIdWithLock(toAccountId).orElseThrow();
    
    // T2: Now trying to lock Account A (but T1 already has it!)
    Account toAccount = accountRepo.findByIdWithLock(fromAccountId).orElseThrow();
    
    fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
    toAccount.setBalance(toAccount.getBalance().add(amount));
    
    accountRepo.save(fromAccount);
    accountRepo.save(toAccount);
}
```

**Deadlock Timeline:**
```
Time    Transaction 1              Transaction 2              Locks
------------------------------------------------------------------------
t1      Lock Account A                                       A: T1
t2                                 Lock Account B            A: T1, B: T2
t3      Try to lock B...           Try to lock A...          
t4      ⏳ WAITING for T2          ⏳ WAITING for T1          
t5      ⏳ WAITING...               ⏳ WAITING...              
t6      💀 DEADLOCK DETECTED!                                
t7      ❌ T1 ROLLED BACK                                    B: T2
t8                                 ✅ T2 continues           B: T2
```

**How to Prevent Deadlocks:**
1. **Lock resources in consistent order**
```java
// Always lock accounts in ascending ID order
@Transactional
public void transferMoneyWithOrder(Long fromId, Long toId, BigDecimal amount) {
    Long firstId = Math.min(fromId, toId);
    Long secondId = Math.max(fromId, toId);
    
    Account first = accountRepo.findByIdWithLock(firstId).orElseThrow();
    Account second = accountRepo.findByIdWithLock(secondId).orElseThrow();
    
    // Now safely do the transfer
}
```

2. **Use timeouts**
```java
@Lock(value = LockModeType.PESSIMISTIC_WRITE, timeout = 5000)  // 5 seconds
```

3. **Minimize lock duration**
4. **Use deadlock detection** (database handles this automatically)

---

### Timeout Issues with PCC

**Problem:** Long-held locks can cause timeouts

```java
@Service
public class ReportService {
    
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void generateHourLongReport(Long accountId) {
        
        // Lock acquired
        Account account = accountRepo.findByIdWithLock(accountId).orElseThrow();
        
        // Long-running operation (1 hour!)
        // Lock held the ENTIRE time
        processComplexReport(account);  // Takes 1 hour
        
        // Other transactions timeout waiting for this lock
        // Lock finally released after 1 hour
    }
}

// Meanwhile, other transactions:
@Transactional
public void quickUpdate(Long accountId) {
    // This will timeout after waiting too long
    Account account = accountRepo.findByIdWithLock(accountId).orElseThrow();
    // Throws: LockTimeoutException after 30 seconds
}
```

**Solutions:**
1. Keep transactions short
2. Use appropriate lock timeout values
3. Consider optimistic locking for long operations
4. Split long transactions into smaller ones

---

### PCC in Distributed Systems

**Distributed Locking with Redis:**
```java
@Service
public class OrderService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    public void processOrderWithDistributedLock(Long orderId) {
        
        // Get distributed lock
        RLock lock = redissonClient.getLock("order:" + orderId);
        
        try {
            // Try to acquire lock (wait up to 10s, auto-release after 30s)
            boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);
            
            if (!acquired) {
                throw new LockAcquisitionException("Could not acquire lock for order: " + orderId);
            }
            
            // Lock acquired - process order
            Order order = orderRepo.findById(orderId).orElseThrow();
            
            // Update inventory service
            inventoryService.decrementStock(order.getProductId(), order.getQuantity());
            
            // Update payment service
            paymentService.processPayment(order.getId(), order.getAmount());
            
            // Update order status
            order.setStatus("COMPLETED");
            orderRepo.save(order);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Lock acquisition interrupted", e);
        } finally {
            // Always release lock
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

---

### When to Use PCC

**✅ Best For:**
- High contention scenarios
- Critical operations where conflicts are expensive
- When retry overhead is unacceptable
- Long-running transactions (but keep them as short as possible)
- Systems where data consistency is critical

**❌ Avoid When:**
- Low contention (unnecessary overhead)
- High throughput requirements
- Distributed systems (deadlocks more likely)
- Cannot afford blocking/waiting

---

## OCC vs PCC Comparison

| Aspect | Optimistic (OCC) | Pessimistic (PCC) |
|--------|------------------|-------------------|
| **Isolation Level** | Below REPEATABLE_READ | REPEATABLE_READ, SERIALIZABLE |
| **Concurrency** | ⚡⚡⚡⚡ Much Higher | ⚡⚡ Less |
| **Performance** | Better with low contention | Better with high contention |
| **Deadlock Risk** | ❌ None | ⚠️ Possible |
| **Conflict Handling** | Detect & retry | Prevent with locks |
| **Overhead** | Retry logic | Lock management, timeouts |
| **Best For** | Read-heavy, distributed | Write-heavy, critical ops |
| **Failure Mode** | OptimisticLockException | LockTimeoutException, Deadlock |
| **Lock Duration** | No locks | Can be long |
| **Scalability** | Better horizontal scaling | Limited by locking |

---

## Complete Summary Diagram

```
CONCURRENCY PROBLEMS:
├── Dirty Read 🔴
│   └── Reading uncommitted data
│       └── Prevented by: READ_COMMITTED+
│
├── Non-Repeatable Read 🟡
│   └── Same row, different values
│       └── Prevented by: REPEATABLE_READ+
│
└── Phantom Read 👻
    └── Different row count
        └── Prevented by: SERIALIZABLE

ISOLATION LEVELS:
├── READ_UNCOMMITTED (L0) ⚡⚡⚡⚡
│   └── No locks, allows all problems
│
├── READ_COMMITTED (L1) ⚡⚡⚡
│   └── Short read locks, prevents dirty reads
│
├── REPEATABLE_READ (L2) ⚡⚡
│   └── Long read locks, prevents non-repeatable reads
│
└── SERIALIZABLE (L3) ⚡
    └── Range locks, prevents all problems

CONCURRENCY CONTROL:
├── Optimistic (OCC)
│   ├── No locks
│   ├── Version checking
│   ├── Retry on conflict
│   └── Best for: Low contention
│
└── Pessimistic (PCC)
    ├── Acquire locks first
    ├── Block conflicting transactions
    ├── Risk of deadlocks
    └── Best for: High contention
```

---

## Quick Reference: Choosing the Right Strategy

### For Traditional Databases (Single Instance)

| Scenario | Recommendation |
|----------|---------------|
| Banking transactions | SERIALIZABLE + Pessimistic Locks |
| E-commerce inventory | REPEATABLE_READ + Optimistic Locks |
| User profile updates | READ_COMMITTED + Optimistic Locks |
| Analytics/Reports | REPEATABLE_READ or SERIALIZABLE |
| High-frequency reads | READ_COMMITTED |
| Rare conflicts | Optimistic Locking |
| Frequent conflicts | Pessimistic Locking |

### For Distributed Systems (Microservices)

| Scenario | Recommendation |
|----------|---------------|
| Order processing | Distributed locks (Redis) + OCC |
| Inventory management | Event sourcing + OCC |
| Payment processing | Saga pattern + PCC |
| User sessions | Optimistic with version |
| Cache updates | Optimistic + TTL |
| Critical shared state | Distributed locks + retry |

---

## Best Practices

1. **Start with lowest isolation level needed** - upgrade only if problems occur
2. **Keep transactions short** - reduces lock contention
3. **Lock in consistent order** - prevents deadlocks
4. **Use appropriate timeouts** - prevents indefinite waiting
5. **Monitor for deadlocks** - add logging and alerting
6. **Implement retry logic** - for optimistic locking failures
7. **Test under load** - concurrency issues appear under stress
8. **Consider eventual consistency** - for distributed systems
9. **Use database features** - let DB handle what it does best
10. **Profile and measure** - don't assume, measure actual performance

---

**End of Notes** ✅
