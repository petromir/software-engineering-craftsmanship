# Backwards-compatibility guide

Table of Contents
-----------------

* [Intro](#intro)
* [API](#api)
    * [Library](#library)
      * [Interfaces](#interfaces)
        * [Adding methods](#adding-methods)
        * [Removing methods](#removing-methods)
        * [Changing method signatures](#changing-method-signatures)
      * [Data classes](#data-classes)
        * [Adding fields](#adding-fields)
        * [Removing fields](#removing-fields)
        * [Changing fields](#changing-fields)
    * [HTTP](#http)
    * [Database](#database)
    * [Events](#events)
      * [Payloads](#payloads)

## Intro

Backwards-compatibility refers to the ability of a system, software, or component to function correctly and as 
intended when new features, updates, or modifications are introduced while still maintaining compatibility with 
older versions or systems. 
By maintaining compatibility with older versions, developers can create more robust, maintainable, 
and scalable software systems.

## API
### Library
#### Interfaces
1. Adding methods

```java
public interface OrderService {
    // Existing method
    void processOrder(Order order);
    
    // Additions
    // A new method with mandatory implementation on a class level
    void createOrder(Order order);
    
    // A new method that does not enforce implementation on a class level. 
    default void validateOrder(Order order) {
        // Implementation details...
    }
}

public class EcommerceOrderService implements OrderService {

    @Override
    void processOrder(final Order order) {
        // Implementation details...
    }
    
    @Override
    void createOrder(final Order order) {
        // Implementation details...
    }
    
    // No need to implement `validateOrder` method` unless we want to override the existing behavior.
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new EcommerceOrderService();
        Order order = new Order();
        orderService.validateOrder(order); // Calling default implementation
        orderService.createOrder(order); // Calling the new method
        orderService.processOrder(order); // Calling the existing method
    }
}
```

2. Removing methods

Removing of methods embraces the `@Deprecated` annotation and its attributes  `since` and `forRemoval`

In our `1.1.0` version we must deprecate the method for removal, but we don't change the class which implements the 
interface.
```java
public interface OrderService {
    
    // Existing method
    @Deprecated(since = "1.1.0", forRemoval = true)
    void processOrder(Order order);
    
    // Existing method
    void createOrder(Order order);
}

// This stays the same
public class EcommerceOrderService implements OrderService {

    @Override
    void processOrder(final Order order) {
        // Implementation details...
    }
    
    @Override
    void createOrder(final Order order) {
        // Implementation details...
    }
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new EcommerceOrderService();
        Order order = new Order();
        orderService.createOrder(order); // Calling the existing method 
        orderService.processOrder(order); // Still able to call the existing deprecated method
    }
}
```

In the next version `2.0.0` we remove the method
```java
public interface OrderService {

    // Leaving only this method
    void createOrder(Order order);
}

// The Previously implemented method is now removed
public class EcommerceOrderService implements OrderService {
    
    @Override
    void createOrder(final Order order) {
        // Implementation details...
    }
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new EcommerceOrderService();
        final Order order = new Order();
        orderService.createOrder(order); // Calling the existing non-deprecated method
    }
}
```

3. Changing method signatures

Just like in the removal process, we embrace the `@Deprecated` annotation and its attributes  `since` and `forRemoval`

In our `1.1.0` version we must deprecate the method with the existing signature, but we don't change the class which 
implements the interface.
```java
public interface OrderService {

    // Existing method
    @Deprecated(since = "1.1.0", forRemoval = true)
    void createOrder(Order order);
    
    // A new method with changed signature
    void createOrder(Order order, Inventory inventory);
    
    // OR A DEFAULT ONE (BUT NOT BOTH AS THE METHOD NAMES WILL CLASH)
    
    // A new method that does not enforce implementation on a class level. 
    default void createOrder(Order order, Inventory inventory) {
        // Implementation details...
    }
}

public class EcommerceOrderService implements OrderService {

    @Override
    void createOrder(final Order order) {
        // Implementation details...
    }
    
    @Override
    void createOrder(final Order order, final Inventory inventory) {
        // Implementation details...
    }
    
    // Or implement the default `createOrder` method if defined and unless the existing behavior must be changed
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new EcommerceOrderService();
        Order order = new Order();
        Inventory inventory = new Inventory();
        orderService.createOrder(order); // Still able to call the existing deprecated method
        orderService.createOrder(order, inventory); // Calling the new method with changed signature
    }
}
```

In the next version `2.0.0` we remove the method with old signature
```java
public interface OrderService {
    	
    // Leaving only the method with changed signature
    void createOrder(Order order, Inventory inventory);
}

public class EcommerceOrderService implements OrderService {

    @Override
    void createOrder(final Order order, final Inventory inventory) {
        // Implementation details...
    }
}

public class Main {
    public static void main(String[] args) {
        OrderService orderService = new EcommerceOrderService();
        Order order = new Order();
        Inventory inventory = new Inventory();
        orderService.createOrder(order, inventory); // Calling the method with changed signature
    }
}
```

#### Data classes
1. Adding fields
```java
@Getter(fluent = true)
public class Order {
    // Existing field
    private String id;
    
    // New field added
    private LocalDateTime createdAt;
}

public class Main {
    public static void main(String[] args) {
        Order order = new Order(1, LocalDateTime.now());
        order.id(); // Accessing the existing field
        order.createdAt(); // Accessing the new field
    }
}
```

2. Removing fields

Removal of the fields embraces the `@Deprecated` annotation and its attributes  `since` and `forRemoval`

In our `1.1.0` version we must deprecate the field for removal.
```java
@Getter(fluent = true)
public class Order {
    // Existing field
    private final String id;
    
    // Existing field
    @Deprecated(since = "1.1.0", forRemoval = true)
    private LocalDateTime createdAt;
}

public class Main {
    public static void main(String[] args) {
        Order order = new Order(1, LocalDateTime.now());
        order.id(); // Accessing the non-deprecated field
        order.createdAt(); // Still able to access the deprecated field
    }
}
```

In the next version `2.0.0` we remove the field.
```java
@Getter(fluent = true)
public class Order {
    // Leaving only this field
    private final String id;
}

public class Main {
    public static void main(String[] args) {
        Order order = new Order(1, LocalDateTime.now());
        order.id(); // Accessing the non-deprecated field
    }
}
```

3. Changing fields

Both changing field type and name go through the same process. Again we are embracing the `@Deprecated` annotation and its attributes  `since` and `forRemoval`

In our `1.1.0` version we must deprecate the field that we would like to change.
```java
@Getter(fluent = true)
public class Order {
    // Existing field
    @Deprecated(since = "1.1.0", forRemoval = true)
    private String orderIdentifier;
    
    // New field with different name and type
    private UUID id;
}

public class Main {
    public static void main(String[] args) {
        Order order = new Order(1, LocalDateTime.now());
        order.id(); // Accessing the new field
        order.orderIdentifier(); // Still able to access the deprecated field 
    }
}
```

In the next version `2.0.0` we remove the old field.
```java
@Getter(fluent = true)
public class Order {
    // Having only one field for identification
    private UUID id;
}

public class Main {
    public static void main(String[] args) {
        Order order = new Order(1, LocalDateTime.now());
        order.id(); // Accessing the new field
    }
}
```

### HTTP

## Database

## Events

### Payloads
