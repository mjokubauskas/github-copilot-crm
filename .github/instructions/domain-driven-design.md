# Domain-Driven Design (DDD) for CRM

Domain-Driven Design (DDD) forms the core of this CRM's architecture by focusing on the business domain and its logic, creating a clean, scalable, and testable design.

## Key Concepts

1. **Entities**:
   - Represents core business objects such as `Customer`, `Order`, `Invoice`, etc.
   - An entity must have a unique `Id` and encapsulate behavior along with data.

   Example:
   ```csharp
   public class Customer
   {
       public Guid Id { get; private set; }
       public string Name { get; private set; }
       public string Email { get; private set; }
       public Address Address { get; private set; }

       private Customer() { }

       public Customer(string name, string email, Address address)
       {
           Id = Guid.NewGuid();
           Name = name;
           Email = email;
           Address = address;
       }

       public void UpdateEmail(string email) {
           Email = email;
       }
   }
   ```
   

2. **Value Objects**:
   - Represent value concepts like `Address` or `Money`.
   - They are immutable and compared by value.

   Example:
   ```csharp
   public record Address(string Street, string City, string ZipCode, string Country);
   ```
   
3. **Aggregates and Roots:**
   - Groups entities that are treated as a single unit of change.
   - Access is only allowed through the root entity (e.g., `Order` contains `OrderItems`, but operations on `OrderItems` must go through the `Order`).

   Example:
   ```csharp
   public class Order
   {
       private List<OrderItem> _items = new();

       public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

       public void AddOrderItem(OrderItem item)
       {
           _items.Add(item);
       }
   }
   ```

4. **Domain Services**:
   - When logic doesn’t fit naturally in an entity or value object.
   - Example: Calculating available inventory.

5. **Domain Events**:
   - Used to notify other parts of the system when something relevant occurs in a domain (event-driven architecture).

   Example:
   ```csharp
   public class OrderCreatedEvent
   {
       public Order Order { get; }

       public OrderCreatedEvent(Order order)
       {
           Order = order;
       }
   }
   ```
