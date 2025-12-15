# Interview Preparation Summary - Java + SQL

## Quick Reference Guide for Tuesday's Interview

---

## 🎯 Core Java Concepts (Must Know)

### **1. OOP Fundamentals**
- **Encapsulation**: Private fields + public getters/setters
- **Inheritance**: `extends` keyword, method overriding
- **Polymorphism**: Interface-based programming
- **Abstraction**: Interfaces and abstract classes

### **2. Collections**
```java
// HashMap - Key-Value pairs
Map<Integer, String> map = new HashMap<>();
map.put(1, "Value");
map.get(1);
map.containsKey(1);

// ArrayList - Dynamic array
List<String> list = new ArrayList<>();
list.add("item");
list.get(0);
list.size();

// Common Operations
- put/get/remove (Map)
- add/get/remove (List)
- containsKey/contains
- size/isEmpty
```

### **3. Java Streams**
```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .collect(Collectors.toList());

list.stream()
    .filter(x -> x.equals(id))
    .findFirst();

list.stream()
    .forEach(System.out::println);
```

### **4. Exception Handling**
```java
try {
    // risky code
} catch (SpecificException e) {
    // handle
} finally {
    // cleanup
}

// Custom Exception
public class CustomException extends Exception { }
```

---

## 🏗️ Design Patterns (Project Implementation)

### **Singleton Pattern**
```java
private static Instance instance;
public static Instance getInstance() {
    if (instance == null) {
        synchronized (Class.class) {
            if (instance == null) {
                instance = new Instance();
            }
        }
    }
    return instance;
}
```
**Why**: Single shared instance, thread-safe, global access

### **Repository Pattern**
- **Interface**: Defines data access methods
- **Implementation**: Concrete data access logic
- **Purpose**: Separate data access from business logic

### **Service Layer Pattern**
- **Business Logic**: Validation, orchestration
- **Coordinates**: Multiple repositories
- **Purpose**: Encapsulate business rules

---

## 📊 SQL Basics (Essential Queries)

### **Basic Operations**
```sql
-- SELECT
SELECT * FROM table WHERE condition;
SELECT col1, col2 FROM table;

-- INSERT
INSERT INTO table (col1, col2) VALUES (val1, val2);

-- UPDATE
UPDATE table SET col1 = val1 WHERE condition;

-- DELETE
DELETE FROM table WHERE condition;
```

### **JOINs**
```sql
-- INNER JOIN
SELECT * FROM t1 JOIN t2 ON t1.id = t2.id;

-- LEFT JOIN
SELECT * FROM t1 LEFT JOIN t2 ON t1.id = t2.id;

-- RIGHT JOIN
SELECT * FROM t1 RIGHT JOIN t2 ON t1.id = t2.id;
```

### **Aggregate Functions**
```sql
SELECT COUNT(*), SUM(price), AVG(price), MIN(price), MAX(price)
FROM table
GROUP BY category
HAVING COUNT(*) > 10;
```

### **Common Patterns**
```sql
-- Second highest
SELECT MAX(price) FROM items WHERE price < (SELECT MAX(price) FROM items);

-- Duplicates
SELECT name, COUNT(*) FROM items GROUP BY name HAVING COUNT(*) > 1;

-- Items never ordered
SELECT * FROM items i LEFT JOIN orders o ON i.id = o.item_id WHERE o.id IS NULL;
```

---

## 🎤 Common Interview Questions

### **Java Questions**

**Q: Explain HashMap vs Hashtable**
- HashMap: Not synchronized, allows null, faster
- Hashtable: Synchronized, no null, thread-safe

**Q: ArrayList vs LinkedList**
- ArrayList: Array-based, fast random access
- LinkedList: Node-based, fast insert/delete

**Q: Checked vs Unchecked Exception**
- Checked: Must handle (IOException, SQLException)
- Unchecked: Optional (NullPointerException, IllegalArgumentException)

**Q: Singleton Pattern - Why double-checked locking?**
- Performance: Avoid locking if instance exists
- Thread-safety: Synchronized block ensures single creation

**Q: Interface vs Abstract Class**
- Interface: Multiple inheritance, only method signatures (Java 8+ default methods)
- Abstract Class: Single inheritance, can have implementation, constructors

### **SQL Questions**

**Q: What is a Primary Key?**
- Unique identifier for each row, cannot be NULL

**Q: What is a Foreign Key?**
- References Primary Key of another table, maintains referential integrity

**Q: Explain JOIN types**
- INNER: Matching records only
- LEFT: All from left + matching from right
- RIGHT: All from right + matching from left
- FULL OUTER: All from both

**Q: GROUP BY vs HAVING**
- GROUP BY: Groups rows
- HAVING: Filters groups (after GROUP BY)

**Q: What is an Index?**
- Improves query performance by creating pointer to data
- Trade-off: Faster SELECT, slower INSERT/UPDATE

---

## 🎯 Project Architecture Overview

```
┌─────────────────────────┐
│  CustomerService        │  ← External API (Customer-facing)
│  StaffService           │  ← External API (Staff-facing)
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│  Business Logic Layer   │
│  - MenuService          │
│  - OrderService         │
│  - BillService          │
│  - TableService         │
│  - ReservationService   │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│  Repository Layer       │
│  - MenuRepository       │  (Singleton)
│  - OrderRepository      │  (Singleton)
│  - TableRepository      │  (Singleton)
│  - BillRepository       │  (Singleton)
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│  Data Storage           │
│  - HashMap              │
│  - (Can be Database)    │
└─────────────────────────┘
```

### **Key Components:**
1. **DTOs**: Data Transfer Objects (MenuItem, Order, Bill, Table)
2. **Enums**: OrderStatus, TableStatus, MenuItemCategory
3. **Exceptions**: BadRequestException
4. **Repositories**: Data access (Singleton pattern)
5. **Services**: Business logic (Dependency Injection)
6. **Response**: Standardized API response format

---

## 📝 Coding Challenge Tips

### **Before You Start:**
1. ✅ Understand requirements clearly
2. ✅ Design architecture (think about layers)
3. ✅ Identify entities and relationships
4. ✅ Plan data structures

### **During Implementation:**
1. ✅ Follow naming conventions
2. ✅ Write clean, readable code
3. ✅ Add comments for complex logic
4. ✅ Handle edge cases
5. ✅ Add proper error handling
6. ✅ Use appropriate data structures

### **Code Structure:**
```
project/
├── dto/          (Data Transfer Objects)
├── enums/        (Enumerations)
├── exceptions/   (Custom Exceptions)
├── repository/   (Data Access Layer)
│   ├── interfaces
│   └── impl/     (Implementations - Singleton)
├── service/      (Business Logic Layer)
│   ├── interfaces
│   └── impl/     (Implementations - Dependency Injection)
└── Main.java     (Entry point, Dependency Wiring)
```

### **Best Practices:**
- ✅ Use interfaces for abstraction
- ✅ Dependency Injection via constructor
- ✅ Single Responsibility Principle
- ✅ Exception handling with custom exceptions
- ✅ Use appropriate collections (HashMap, ArrayList)
- ✅ Thread-safety where needed (Singleton)

---

## 🚀 Quick SQL Practice Queries

### **For Restaurant Management:**

```sql
-- Total revenue today
SELECT SUM(total_amount) 
FROM bills 
WHERE DATE(payment_time) = CURDATE() AND payment_status = 'PAID';

-- Most popular item
SELECT m.name, SUM(oi.quantity) as total_quantity
FROM order_items oi
JOIN menu_items m ON oi.menu_id = m.menu_id
GROUP BY m.menu_id, m.name
ORDER BY total_quantity DESC
LIMIT 1;

-- Average order value
SELECT AVG(total_amount) as avg_order_value
FROM bills
WHERE payment_status = 'PAID';

-- Items never ordered
SELECT m.*
FROM menu_items m
LEFT JOIN order_items oi ON m.menu_id = oi.menu_id
WHERE oi.menu_id IS NULL;

-- Revenue by category
SELECT m.category, SUM(oi.quantity * oi.price) as revenue
FROM order_items oi
JOIN menu_items m ON oi.menu_id = m.menu_id
JOIN orders o ON oi.order_id = o.order_id
JOIN bills b ON o.order_id = b.order_id
WHERE b.payment_status = 'PAID'
GROUP BY m.category;
```

---

## ✅ Checklist Before Interview

### **Java Knowledge:**
- [ ] OOP concepts (Encapsulation, Inheritance, Polymorphism, Abstraction)
- [ ] Collections (HashMap, ArrayList, operations)
- [ ] Java Streams (filter, map, collect, findFirst)
- [ ] Exception Handling (try-catch, custom exceptions)
- [ ] Interfaces and Abstract Classes
- [ ] Access Modifiers (private, protected, public)
- [ ] Static vs Instance

### **Design Patterns:**
- [ ] Singleton Pattern (double-checked locking)
- [ ] Repository Pattern (why and how)
- [ ] Service Layer Pattern (business logic)
- [ ] Dependency Injection (constructor injection)

### **SQL Knowledge:**
- [ ] DDL (CREATE, ALTER, DROP)
- [ ] DML (INSERT, UPDATE, DELETE, SELECT)
- [ ] JOINs (INNER, LEFT, RIGHT)
- [ ] Aggregate Functions (COUNT, SUM, AVG, MIN, MAX)
- [ ] GROUP BY and HAVING
- [ ] Subqueries
- [ ] Constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL)
- [ ] Indexes (when and why)

### **Project Understanding:**
- [ ] Understand your LLD project architecture
- [ ] Can explain design decisions
- [ ] Know how to extend the project
- [ ] Understand data flow
- [ ] Can identify improvements

---

## 💡 Last-Minute Tips

1. **Be Confident**: You've built a complete system!
2. **Explain Your Choices**: Why Singleton? Why Repository Pattern?
3. **Think Aloud**: During coding, explain your thought process
4. **Ask Questions**: Clarify requirements before coding
5. **Start Simple**: Don't overcomplicate initially
6. **Test Your Code**: Think about edge cases
7. **Clean Code**: Follow naming conventions, proper structure
8. **Time Management**: Allocate time wisely (design → implement → test)

---

## 🎓 Resources for Quick Revision

1. **Java Concepts**: `JAVA_CONCEPTS_LEARNING.md`
2. **SQL Basics**: `SQL_BASICS_LEARNING.md`
3. **Design Patterns**: `DESIGN_PATTERNS_EXPLAINED.md`
4. **Project Code**: Review your implementation

---

## 📌 Key Takeaways

### **Java:**
- OOP principles
- Collections and Streams
- Exception handling
- Design patterns

### **SQL:**
- CRUD operations
- JOINs
- Aggregate functions
- Query optimization

### **Architecture:**
- Layered architecture
- Separation of concerns
- Dependency Injection
- Design patterns

**Good luck with your interview! You've got this! 🚀**

