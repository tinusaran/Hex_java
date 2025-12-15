# Design Patterns in Restaurant Management LLD Project

## Table of Contents
1. [Singleton Pattern](#singleton-pattern)
2. [Repository Pattern](#repository-pattern)
3. [Service Layer Pattern](#service-layer-pattern)
4. [Strategy Pattern](#strategy-pattern)
5. [Factory Pattern](#factory-pattern)
6. [Dependency Injection](#dependency-injection)

---

## Singleton Pattern

### **Purpose:**
Ensure only one instance of a class exists in the application lifecycle.

### **Problem it Solves:**
- Need shared state across the application
- Expensive object creation (database connections, cache)
- Global access point

### **Implementation in Project:**

```java
// MenuRepositoryImpl.java
public class MenuRepositoryImpl implements IMenuRepository {
    private static IMenuRepository instance;  // Static instance
    private final Map<Integer, MenuItem> menuItems = new HashMap<>();
    
    // Private constructor - prevents external instantiation
    private MenuRepositoryImpl() {
        // Initialization code
    }
    
    // Public static method to get instance
    public static MenuRepositoryImpl getInstance() {
        if (instance == null) {  // First check (avoid locking if exists)
            synchronized (IMenuRepository.class) {  // Thread-safe
                if (instance == null) {  // Second check (double-checked locking)
                    instance = new MenuRepositoryImpl();
                }
            }
        }
        return (MenuRepositoryImpl) instance;
    }
}
```

### **Why Double-Checked Locking?**
1. **Performance**: Avoids synchronization overhead if instance exists
2. **Thread Safety**: Synchronized block ensures only one thread creates instance
3. **Correctness**: Second check prevents multiple instances

### **Usage:**
```java
IMenuRepository repo1 = MenuRepositoryImpl.getInstance();
IMenuRepository repo2 = MenuRepositoryImpl.getInstance();
// repo1 == repo2 (same instance)
```

### **Interview Points:**
- ✅ Thread-safe implementation
- ✅ Lazy initialization
- ✅ Prevents multiple instances
- ✅ Global access point

### **Alternatives:**
```java
// Eager initialization (thread-safe, but always creates instance)
private static final MenuRepositoryImpl instance = new MenuRepositoryImpl();

// Enum singleton (best for thread-safety)
public enum MenuRepository {
    INSTANCE;
    // methods
}
```

---

## Repository Pattern

### **Purpose:**
Separate data access logic from business logic. Acts as a mediator between business logic and data source.

### **Structure:**
```
┌─────────────────┐
│  Service Layer  │
└────────┬────────┘
         │
         │ uses
         ▼
┌─────────────────┐
│  Repository     │  (Interface)
│  Interface      │
└────────┬────────┘
         │
         │ implements
         ▼
┌─────────────────┐
│  Repository     │
│  Implementation │
└────────┬────────┘
         │
         │ accesses
         ▼
┌─────────────────┐
│  Data Source    │  (HashMap, Database, File, etc.)
└─────────────────┘
```

### **Implementation in Project:**

**1. Interface (Abstraction):**
```java
public interface IMenuRepository {
    MenuItem addMenuItem(MenuItem menuItem) throws BadRequestException;
    MenuItem updateMenuItem(MenuItem menuItem) throws BadRequestException;
    MenuItem getMenuItem(Integer id, String name) throws BadRequestException;
    List<MenuItem> getMenuItems() throws BadRequestException;
}
```

**2. Implementation:**
```java
public class MenuRepositoryImpl implements IMenuRepository {
    private final Map<Integer, MenuItem> menuItems = new HashMap<>();
    private final Map<String, Integer> reverseIndexing = new HashMap<>();
    
    @Override
    public MenuItem addMenuItem(MenuItem menuItem) {
        menuItems.put(menuItem.getMenuId(), menuItem);
        reverseIndexing.put(menuItem.getName(), menuItem.getMenuId());
        return menuItem;
    }
    
    @Override
    public MenuItem getMenuItem(Integer id, String name) throws BadRequestException {
        if (Objects.nonNull(name)) {
            id = reverseIndexing.get(name);  // Lookup by name
        }
        if (!menuItems.containsKey(id)) {
            throw new BadRequestException("Item Not Found");
        }
        return menuItems.get(id);
    }
}
```

**3. Usage in Service:**
```java
public class MenuServiceImpl implements IMenuService {
    private final IMenuRepository menuRepository;  // Depends on interface
    
    public MenuServiceImpl(IMenuRepository menuRepository) {
        this.menuRepository = menuRepository;  // Dependency Injection
    }
    
    @Override
    public MenuItem addMenuItem(...) throws BadRequestException {
        MenuItem menuItem = new MenuItem(id, name, price, category);
        return menuRepository.addMenuItem(menuItem);  // Delegate to repository
    }
}
```

### **Benefits:**
1. **Separation of Concerns**: Business logic doesn't know about data storage
2. **Testability**: Easy to mock repository for testing
3. **Flexibility**: Can swap implementations (HashMap → Database → File)
4. **Single Responsibility**: Repository only handles data access
5. **Abstraction**: Service layer works with interface, not implementation

### **Future Extension (Database):**
```java
public class MenuRepositoryDBImpl implements IMenuRepository {
    private final Connection connection;
    
    @Override
    public MenuItem addMenuItem(MenuItem menuItem) {
        // SQL: INSERT INTO menu_items ...
    }
    
    @Override
    public MenuItem getMenuItem(Integer id, String name) {
        // SQL: SELECT * FROM menu_items WHERE ...
    }
}
// No changes needed in Service layer!
```

### **Interview Points:**
- ✅ Data access abstraction
- ✅ Testability through mocking
- ✅ Single Responsibility Principle
- ✅ Open/Closed Principle (extend without modifying)

---

## Service Layer Pattern

### **Purpose:**
Contains business logic and orchestrates multiple repositories. Acts as an intermediary between controllers/APIs and repositories.

### **Structure:**
```
┌─────────────────┐
│   Controller    │  (API/UI Layer)
│     / API       │
└────────┬────────┘
         │
         │ calls
         ▼
┌─────────────────┐
│  Service Layer  │  (Business Logic)
└────────┬────────┘
         │
         │ uses
         ▼
┌─────────────────┐
│  Repository     │
│    Layer        │
└─────────────────┘
```

### **Implementation in Project:**

**1. Interface:**
```java
public interface IMenuService {
    MenuItem addMenuItem(Integer id, String name, double price, MenuItemCategory category) 
        throws BadRequestException;
    MenuItem updateMenuItemPrice(Integer id, double newPrice) throws BadRequestException;
    List<MenuItem> showMenu() throws BadRequestException;
}
```

**2. Implementation:**
```java
public class MenuServiceImpl implements IMenuService {
    private final IMenuRepository menuRepository;  // Dependency
    
    // Dependency Injection via constructor
    public MenuServiceImpl(IMenuRepository menuRepository) {
        this.menuRepository = menuRepository;
    }
    
    @Override
    public MenuItem addMenuItem(Integer id, String name, double price, MenuItemCategory category) 
            throws BadRequestException {
        // Business Logic: Validation, transformation
        MenuItem menuItem = new MenuItem(id, name, price, category);
        
        // Delegate to repository
        return menuRepository.addMenuItem(menuItem);
    }
    
    @Override
    public MenuItem updateMenuItemPrice(Integer id, double newPrice) throws BadRequestException {
        // Business Logic: Get existing, validate, update
        MenuItem menuItem = menuRepository.getMenuItem(id, null);
        if (menuItem == null) {
            throw new BadRequestException("MenuItem not found");
        }
        menuItem.setPrice(newPrice);
        return menuRepository.updateMenuItem(menuItem);
    }
}
```

**3. Orchestrating Multiple Repositories:**
```java
public class OrderServiceImpl implements IOrderService {
    private final IOrderRepository orderRepository;
    private final IMenuRepository menuRepository;  // Multiple dependencies
    
    public OrderServiceImpl(IOrderRepository orderRepository, IMenuRepository menuRepository) {
        this.orderRepository = orderRepository;
        this.menuRepository = menuRepository;
    }
    
    @Override
    public Orders addItemInOrder(Integer customerId, String name, Integer quantity) 
            throws BadRequestException {
        // Business Logic: Get menu item, validate, add to order
        MenuItem menuItem = menuRepository.getMenuItem(null, name);
        
        List<Orders> orders = orderRepository.getOrder(customerId, OrderStatus.CREATED);
        if (orders == null || orders.isEmpty()) {
            throw new BadRequestException("Order not found");
        }
        
        Orders order = orders.get(0);
        order.getOrderItems().add(new OrderItem(...));
        
        return orderRepository.addOrderItem(order);
    }
}
```

### **Benefits:**
1. **Business Logic Centralization**: All business rules in one place
2. **Reusability**: Services can be used by multiple controllers
3. **Transaction Management**: Can handle transactions across repositories
4. **Validation**: Centralized validation logic
5. **Orchestration**: Coordinates multiple repositories

### **Layered Architecture:**
```
┌─────────────────────────────────┐
│   Presentation Layer (API/UI)   │  ← CustomerService, StaffService
└──────────────┬──────────────────┘
               │
┌──────────────▼──────────────────┐
│   Business Logic Layer          │  ← MenuService, OrderService, BillService
│   (Service Layer)               │
└──────────────┬──────────────────┘
               │
┌──────────────▼──────────────────┐
│   Data Access Layer             │  ← MenuRepository, OrderRepository
│   (Repository Layer)            │
└──────────────┬──────────────────┘
               │
┌──────────────▼──────────────────┐
│   Data Storage                  │  ← HashMap, Database, File
└─────────────────────────────────┘
```

### **Interview Points:**
- ✅ Separation of concerns
- ✅ Business logic encapsulation
- ✅ Dependency Injection
- ✅ Testability
- ✅ Scalability

---

## Dependency Injection (DI)

### **Purpose:**
Provide dependencies to a class from outside, rather than creating them internally.

### **Problem it Solves:**
```java
// ❌ Bad: Tight coupling
public class MenuServiceImpl {
    private IMenuRepository menuRepository = new MenuRepositoryImpl();  // Hard dependency
}

// ✅ Good: Dependency Injection
public class MenuServiceImpl {
    private final IMenuRepository menuRepository;
    
    public MenuServiceImpl(IMenuRepository menuRepository) {  // Injected dependency
        this.menuRepository = menuRepository;
    }
}
```

### **Implementation in Project:**
```java
// Main.java - Dependency Injection Container (manual)
public class Main {
    public static void main(String[] args) {
        // Create dependencies
        IMenuRepository menuRepository = MenuRepositoryImpl.getInstance();
        
        // Inject dependencies
        IMenuService menuService = new MenuServiceImpl(menuRepository);
        
        // Use service
        ICustomerService customerService = new CustomerServiceImpl(
            menuService, orderService, reservationService, billService
        );
    }
}
```

### **Types of Dependency Injection:**
1. **Constructor Injection** (used in project):
```java
public MenuServiceImpl(IMenuRepository menuRepository) {
    this.menuRepository = menuRepository;
}
```

2. **Setter Injection:**
```java
public void setMenuRepository(IMenuRepository menuRepository) {
    this.menuRepository = menuRepository;
}
```

3. **Field Injection** (Spring Boot):
```java
@Autowired
private IMenuRepository menuRepository;
```

### **Benefits:**
- ✅ Loose coupling
- ✅ Testability (easy to mock)
- ✅ Flexibility (swap implementations)
- ✅ Single Responsibility

---

## Strategy Pattern (Conceptual)

### **Purpose:**
Define a family of algorithms, encapsulate each one, and make them interchangeable.

### **Example:**
Different payment strategies:
```java
// Strategy Interface
public interface PaymentStrategy {
    void pay(double amount);
}

// Concrete Strategies
public class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) { /* credit card logic */ }
}

public class CashPayment implements PaymentStrategy {
    public void pay(double amount) { /* cash logic */ }
}

// Context
public class BillService {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void processPayment(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

### **In Project Context:**
Currently payment method is a String. Could be refactored to Strategy pattern for different payment processors.

---

## Factory Pattern (Conceptual)

### **Purpose:**
Create objects without specifying the exact class of object that will be created.

### **Example:**
```java
public class RepositoryFactory {
    public static IMenuRepository createMenuRepository(String type) {
        if ("memory".equals(type)) {
            return MenuRepositoryImpl.getInstance();
        } else if ("database".equals(type)) {
            return new MenuRepositoryDBImpl();
        }
        throw new IllegalArgumentException("Unknown type: " + type);
    }
}
```

---

## Design Principles Applied

### **1. SOLID Principles:**

**S - Single Responsibility Principle:**
- Repository: Only data access
- Service: Only business logic
- DTO: Only data structure

**O - Open/Closed Principle:**
- Can extend with new repository implementations without modifying service

**L - Liskov Substitution Principle:**
- Any implementation of IMenuRepository can replace another

**I - Interface Segregation Principle:**
- Interfaces are focused and specific

**D - Dependency Inversion Principle:**
- High-level modules depend on abstractions (interfaces), not concretions

### **2. DRY (Don't Repeat Yourself):**
- Shared logic in services
- Common utilities in repositories

### **3. Separation of Concerns:**
- Clear boundaries between layers
- Each layer has specific responsibility

---

## Interview Questions & Answers

### **Q1: Why use Singleton for Repository?**
**A:** Repositories manage shared state (in-memory storage). Singleton ensures:
- Single source of truth
- Consistent data across services
- Efficient memory usage

### **Q2: Why Repository Pattern instead of direct data access?**
**A:** 
- Separation of concerns
- Easy to swap data sources
- Testability (mock repositories)
- Centralized data access logic

### **Q3: Why Service Layer?**
**A:**
- Encapsulates business logic
- Orchestrates multiple repositories
- Reusable across different interfaces
- Transaction management

### **Q4: What's the difference between Repository and Service?**
**A:**
- **Repository**: Data access, CRUD operations, no business logic
- **Service**: Business logic, validation, orchestration, coordinates repositories

### **Q5: How would you test these patterns?**
**A:**
- Mock repositories for service testing
- Use dependency injection to inject mocks
- Test business logic in isolation
- Integration tests for full flow

---

## Summary

| Pattern | Purpose | Used In |
|---------|---------|---------|
| Singleton | Single instance | Repositories |
| Repository | Data access abstraction | All repositories |
| Service Layer | Business logic | All services |
| Dependency Injection | Loose coupling | Constructor injection |
| Strategy | Interchangeable algorithms | Payment (potential) |
| Factory | Object creation | Repository creation (potential) |

These patterns create a **maintainable, testable, and scalable** architecture!

