# Java Concepts Used in Restaurant Management LLD Project

## Table of Contents
1. [Object-Oriented Programming (OOP)](#object-oriented-programming-oop)
2. [Collections Framework](#collections-framework)
3. [Java Streams API](#java-streams-api)
4. [Exception Handling](#exception-handling)
5. [Interfaces and Abstraction](#interfaces-and-abstraction)
6. [Singleton Pattern](#singleton-pattern)
7. [Repository Pattern](#repository-pattern)
8. [Service Layer Pattern](#service-layer-pattern)

---

## Object-Oriented Programming (OOP)

### 1. **Encapsulation**
Encapsulation means bundling data and methods together, hiding internal details.

**Example from the project:**
```java
// MenuItem.java - Encapsulation in action
public class MenuItem {
    private Integer menuId;  // Private field
    private String name;
    private double price;
    
    // Public getters and setters
    public Integer getMenuId() { return menuId; }
    public void setMenuId(Integer menuId) { this.menuId = menuId; }
}
```

**Key Points:**
- Fields are `private` to prevent direct access
- Access through public getters/setters
- Maintains data integrity and control

### 2. **Inheritance**
Inheritance allows classes to inherit properties and methods from parent classes.

**Example:**
```java
// Custom Exception
public class BadRequestException extends Exception {
    private String message;
    // Inherits all Exception methods (getMessage, printStackTrace, etc.)
}
```

### 3. **Polymorphism**
Polymorphism allows objects of different classes to be treated through the same interface.

**Example:**
```java
// Interface
IMenuRepository menuRepository = MenuRepositoryImpl.getInstance();
// Can be replaced with any implementation of IMenuRepository
```

### 4. **Abstraction**
Abstraction hides implementation details, showing only essential features.

**Example:**
```java
// Interface abstracts the implementation
public interface IMenuService {
    MenuItem addMenuItem(Integer id, String name, double price, MenuItemCategory category);
    // Implementation details hidden
}
```

---

## Collections Framework

### 1. **HashMap**
Stores key-value pairs, allows fast lookup.

**Used in the project:**
```java
// MenuRepositoryImpl.java
private final Map<Integer, MenuItem> menuItems = new HashMap<>();
private final Map<String, Integer> reverseIndexing = new HashMap<>();

// Operations:
menuItems.put(menuItem.getMenuId(), menuItem);  // Add
MenuItem item = menuItems.get(id);              // Get
menuItems.containsKey(id);                      // Check existence
```

**Key Methods:**
- `put(key, value)` - Add/Update
- `get(key)` - Retrieve
- `containsKey(key)` - Check existence
- `remove(key)` - Remove
- `values()` - Get all values
- `keySet()` - Get all keys

### 2. **ArrayList**
Dynamic array that can grow/shrink.

**Used in the project:**
```java
// Orders.java
private List<OrderItem> orderItems;  // Can be ArrayList

// Operations:
orderItems.add(newItem);             // Add
orderItems.get(index);               // Get by index
orderItems.remove(index);            // Remove
orderItems.size();                   // Get size
orderItems.isEmpty();                // Check if empty
```

**Key Methods:**
- `add(element)` - Add to end
- `get(index)` - Get by index
- `remove(index)` - Remove by index
- `size()` - Get size
- `isEmpty()` - Check if empty
- `contains(element)` - Check existence

### 3. **Common Collection Operations**
```java
// Convert HashMap values to ArrayList
List<MenuItem> menuList = new ArrayList<>(menuItems.values());

// Iterate over HashMap
for (Map.Entry<Integer, MenuItem> entry : menuItems.entrySet()) {
    Integer key = entry.getKey();
    MenuItem value = entry.getValue();
}

// Iterate over List
for (MenuItem item : menuList) {
    System.out.println(item.getName());
}
```

---

## Java Streams API

Streams allow functional-style operations on collections.

### **Example from OrderRepositoryImpl:**
```java
// Filter orders by status
return allCusOrders.stream()
    .filter(order -> order.getStatus().equals(status))
    .collect(Collectors.toList());
```

### **Common Stream Operations:**

1. **filter()** - Filter elements based on condition
```java
list.stream()
    .filter(item -> item.getPrice() > 100)
    .collect(Collectors.toList());
```

2. **map()** - Transform elements
```java
list.stream()
    .map(item -> item.getName().toUpperCase())
    .collect(Collectors.toList());
```

3. **findFirst()** - Get first matching element
```java
Optional<MenuItem> item = list.stream()
    .filter(m -> m.getMenuId().equals(id))
    .findFirst();
```

4. **forEach()** - Perform action on each element
```java
list.stream()
    .forEach(item -> System.out.println(item));
```

5. **collect()** - Collect results into collection
```java
List<String> names = list.stream()
    .map(MenuItem::getName)
    .collect(Collectors.toList());
```

6. **anyMatch() / allMatch()** - Check conditions
```java
boolean hasExpensive = list.stream()
    .anyMatch(item -> item.getPrice() > 100);
```

---

## Exception Handling

### **Types of Exceptions:**

1. **Checked Exceptions** (must be handled)
```java
public class BadRequestException extends Exception { }
```

2. **Unchecked Exceptions** (RuntimeException)
```java
// NullPointerException, IllegalArgumentException, etc.
```

### **Exception Handling Patterns:**

**1. Try-Catch Block:**
```java
try {
    MenuItem item = menuService.addMenuItem(1, "Pizza", 100.0, category);
    return new Response(item, "OK", 200);
} catch (BadRequestException e) {
    return new Response(null, e.getMessage(), 400);
}
```

**2. Throws Declaration:**
```java
public MenuItem addMenuItem(...) throws BadRequestException {
    if (id == null) {
        throw new BadRequestException("ID cannot be null");
    }
    // ...
}
```

**3. Custom Exception:**
```java
public class BadRequestException extends Exception {
    private String message;
    
    public BadRequestException(String message) {
        this.message = message;
    }
    
    @Override
    public String getMessage() {
        return message;
    }
}
```

### **Best Practices:**
- Use specific exceptions (BadRequestException vs generic Exception)
- Provide meaningful error messages
- Handle exceptions at appropriate levels
- Don't catch and ignore exceptions

---

## Interfaces and Abstraction

### **Interface Definition:**
```java
public interface IMenuService {
    MenuItem addMenuItem(Integer id, String name, double price, MenuItemCategory category) 
        throws BadRequestException;
    List<MenuItem> showMenu() throws BadRequestException;
}
```

### **Implementation:**
```java
public class MenuServiceImpl implements IMenuService {
    private final IMenuRepository menuRepository;
    
    @Override
    public MenuItem addMenuItem(...) throws BadRequestException {
        // Implementation
    }
}
```

### **Benefits:**
- **Loose Coupling**: Code depends on interface, not implementation
- **Testability**: Easy to mock interfaces for testing
- **Flexibility**: Can swap implementations easily
- **Multiple Inheritance**: Class can implement multiple interfaces

### **Usage in Project:**
```java
// Dependency Injection through constructor
public MenuServiceImpl(IMenuRepository menuRepository) {
    this.menuRepository = menuRepository;
}
```

---

## Singleton Pattern

### **Purpose:**
Ensure only one instance of a class exists throughout the application.

### **Implementation (Double-Checked Locking):**
```java
public class MenuRepositoryImpl implements IMenuRepository {
    private static IMenuRepository instance;
    
    private MenuRepositoryImpl() {
        // Private constructor prevents direct instantiation
    }
    
    public static MenuRepositoryImpl getInstance() {
        if (instance == null) {
            synchronized (IMenuRepository.class) {
                if (instance == null) {
                    instance = new MenuRepositoryImpl();
                }
            }
        }
        return (MenuRepositoryImpl) instance;
    }
}
```

### **Why Double-Checked Locking?**
- First check: Avoid locking if instance exists
- Synchronized block: Thread-safe creation
- Second check: Ensure only one instance is created

### **Usage:**
```java
IMenuRepository repository = MenuRepositoryImpl.getInstance();
// Always returns the same instance
```

---

## Repository Pattern

### **Purpose:**
Separate data access logic from business logic.

### **Structure:**
```
Interface (IMenuRepository)
    ↓
Implementation (MenuRepositoryImpl)
    ↓
Data Storage (HashMap, Database, etc.)
```

### **Example:**
```java
// Interface
public interface IMenuRepository {
    MenuItem addMenuItem(MenuItem menuItem) throws BadRequestException;
    MenuItem getMenuItem(Integer id, String name) throws BadRequestException;
    List<MenuItem> getMenuItems() throws BadRequestException;
}

// Implementation
public class MenuRepositoryImpl implements IMenuRepository {
    private final Map<Integer, MenuItem> menuItems = new HashMap<>();
    
    @Override
    public MenuItem addMenuItem(MenuItem menuItem) {
        menuItems.put(menuItem.getMenuId(), menuItem);
        return menuItem;
    }
}
```

### **Benefits:**
- Centralizes data access logic
- Easy to swap data sources (HashMap → Database)
- Improves testability
- Single Responsibility Principle

---

## Service Layer Pattern

### **Purpose:**
Contains business logic, orchestrates multiple repositories.

### **Structure:**
```
Controller/API Layer
    ↓
Service Layer (Business Logic)
    ↓
Repository Layer (Data Access)
```

### **Example:**
```java
public class MenuServiceImpl implements IMenuService {
    private final IMenuRepository menuRepository;  // Dependency
    
    public MenuServiceImpl(IMenuRepository menuRepository) {
        this.menuRepository = menuRepository;  // Dependency Injection
    }
    
    @Override
    public MenuItem addMenuItem(Integer id, String name, double price, MenuItemCategory category) 
            throws BadRequestException {
        // Business Logic: Validation, transformation, etc.
        MenuItem menuItem = new MenuItem(id, name, price, category);
        
        // Delegate to repository
        return menuRepository.addMenuItem(menuItem);
    }
}
```

### **Benefits:**
- Separation of concerns
- Business logic centralized
- Easy to test
- Reusable across different interfaces

---

## Key Java Concepts Summary

### **1. Access Modifiers:**
- `private` - Only within class
- `protected` - Within package + subclasses
- `default` (no modifier) - Within package
- `public` - Everywhere

### **2. Keywords:**
- `static` - Belongs to class, not instance
- `final` - Cannot be changed (variable) or extended (class)
- `this` - Reference to current object
- `super` - Reference to parent class

### **3. Object Methods:**
- `toString()` - String representation
- `equals()` - Equality check
- `hashCode()` - Hash code for collections

### **4. Null Safety:**
```java
// Check for null
if (Objects.isNull(object)) { }
if (Objects.nonNull(object)) { }

// Optional (Java 8+)
Optional<String> name = Optional.ofNullable(value);
name.ifPresent(System.out::println);
```

---

## Interview Tips

### **Common Questions:**

1. **What is the difference between HashMap and Hashtable?**
   - HashMap: Not synchronized, allows null keys/values, faster
   - Hashtable: Synchronized, no null keys/values, thread-safe

2. **Explain ArrayList vs LinkedList?**
   - ArrayList: Array-based, fast random access, slow insertion/deletion
   - LinkedList: Node-based, slow random access, fast insertion/deletion

3. **What is the difference between checked and unchecked exceptions?**
   - Checked: Must be handled (IOException, SQLException)
   - Unchecked: Need not be handled (NullPointerException, IllegalArgumentException)

4. **Explain Singleton pattern and thread-safety?**
   - Double-checked locking ensures thread-safe singleton creation

5. **What is the difference between interface and abstract class?**
   - Interface: Only method signatures, multiple inheritance, Java 8+ default methods
   - Abstract class: Can have implementation, single inheritance, can have constructors

---

## Practice Exercises

1. Implement a method to find menu items by category using streams
2. Add validation to prevent duplicate menu items
3. Implement a method to calculate total revenue from all paid bills
4. Add a method to get all free tables using streams
5. Implement error handling for all edge cases

