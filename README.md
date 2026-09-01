# OOP is a mistake. Simplicity is key.

The central claim is simple: OOP often breaks related logic apart, spreads state across too many places, and makes programs harder to understand as they grow.

## 1. The core OOP promise

Traditional OOP teaches that software should be composed of objects. Each object combines:

- Data: the object’s current state
- Behavior: methods that operate on that state
- Encapsulation: hiding internal details
- Inheritance and polymorphism: sharing or substituting behavior between types

For example, an `Order` object might contain its items, customer, status, and total. It may also expose methods such as `addItem()`, `cancel()`, or `calculateTotal()`.

This feels intuitive because we think in nouns: customers, orders, messages, accounts, documents. OOP says the code should mirror those nouns.

*The problem is that software is usually more about processes than nouns.*

## 2. Real business logic crosses object boundaries

Consider a simple operation: placing an order.

It may need to:

1. Validate the customer
2. Check product availability
3. Calculate discounts and tax
4. Charge a payment method
5. Reserve inventory
6. Create a shipment
7. Send a confirmation
8. Record analytics

Which object should own that behavior?

If `Order` owns it, then the `Order` class may need to know about payments, inventory, shipping, email, and analytics. It becomes a “god object.”

If each object manages its own part, the logic becomes scattered. `Order` calls `Payment`, which calls `Inventory`, which triggers `Shipment`, which notifies `EmailService`. The business process is now distributed across many classes.

Neither approach is ideal. The key workflow—“place an order”—is no longer visible in one place. A developer trying to change it must trace calls across the system.

This is the fragmentation problem: OOP organizes code around data containers, while important behavior often concerns several containers at once.

## 3. Encapsulation can hide the information you need

Encapsulation is often presented as an unquestionable good: objects should hide their internal state and expose only methods.

For a small implementation detail, this is useful. A `DatabaseConnection` should hide its socket handling. A `HashMap` should hide its bucket layout.

But applying the same rule to every piece of application data can become counterproductive.

Imagine a `Customer` object that hides its address, payment status, account tier, and purchase history behind many getter methods or specialized operations. Then an `OrderService` needs to understand the customer in order to make a decision—but the relevant information is concealed behind a narrow interface designed in advance.

Developers often respond by adding more getters, setters, helper methods, and cross-object calls. The supposed boundary remains, but the code on either side is tightly coupled anyway.

A useful rule is:

> Hide implementation details, not useful facts.

Data that is naturally shared across a business process should not be artificially difficult to access merely because it belongs to an object.

## 4. Mutable object state creates invisible dependencies

Many OOP systems are built around objects that change over time.

```text
order.addItem(product)
order.applyCoupon(code)
order.calculateTax()
order.chargeCard(card)
order.markPaid()
order.ship()
```

Each method appears simple. But the correctness of each call depends on the object’s hidden state and the calls that happened before it.

What happens if `ship()` is called before `markPaid()`? What if a coupon is applied after tax was calculated? What if payment succeeds but inventory reservation fails?

The possible states and transitions multiply quickly. The program’s behavior becomes dependent on history, not just on its visible inputs.

Compare that with a more explicit model:

```text
validatedOrder = validate(order)
pricedOrder = calculatePrice(validatedOrder, catalog, discounts)
paidOrder = charge(pricedOrder, paymentMethod)
fulfilledOrder = fulfill(paidOrder, inventory)
```

Here each stage receives data and produces a new result. Dependencies are easier to see. Each operation can be tested without constructing an elaborate object graph or reproducing a particular sequence of mutations.

The point is not that mutation is forbidden. It is that mutable state should be treated as a cost. OOP often normalizes that cost by placing mutable state inside nearly every object.

## 5. Inheritance makes change expensive

Inheritance is another major OOP feature that sounds useful in theory.

```text
Employee
├── FullTimeEmployee
├── Contractor
└── Intern
```

At first, this hierarchy appears clean. Then the real world intrudes:

- Some contractors receive benefits.
- Some full-time employees are paid hourly.
- Interns may become contractors.
- Payroll rules differ by country.
- A person may hold multiple roles.

The hierarchy starts requiring exceptions, overrides, flags, interfaces, and special subclasses. The model becomes less about the real domain and more about preserving the class structure.

Inheritance also creates hidden coupling: a child class depends on the behavior and internal assumptions of its parent. A change to a base class can unintentionally affect every subclass.

Composition—combining smaller, focused capabilities—is usually more flexible. Rather than making every person fit into one permanent subtype, store explicit data about roles, compensation policies, permissions, and benefits, then use functions or services to apply the relevant rules.

## 6. Design patterns often reveal missing language features

Large OOP codebases often include patterns such as factories, visitors, builders, adapters, strategies, commands, and dependency injection containers.

These patterns are not inherently bad. Some are useful and necessary in certain designs. But a system that relies on many layers of patterns may be compensating for the constraints of organizing everything as classes.

For example, a factory may exist because constructors cannot conveniently represent the different ways an object must be created. A visitor may exist because a class hierarchy makes it awkward to add a new operation across several types. A strategy may exist because behavior must vary, but inheritance is too rigid.

The question should not be “which pattern applies?” first. It should be:

> Is this complexity required by the problem, or by the object model we chose?

Often, a plain data structure and a direct function are easier to understand than a family of interfaces and implementations.

## 7. Organize code around operations

The alternative presented by the critique is usually a more procedural or data-oriented style.

That does not mean writing one enormous file full of unrelated functions. It means grouping code by meaningful operations and keeping the data model straightforward.

Instead of this:

```text
order.place()
payment.process()
inventory.reserve()
shipment.create()
```

A workflow-oriented design might use:

```text
placeOrder(orderRequest, customer, catalog, inventory, paymentGateway)
```

Inside that operation, the full business process is visible. Supporting code can still be split into focused modules:

```text
pricing.calculate(...)
inventory.reserve(...)
payments.charge(...)
shipping.create(...)
notifications.send(...)
```

This approach has several advantages:

- The main workflow is easy to locate.
- Dependencies are visible in function parameters.
- Data can move through the system without being hidden inside objects.
- Business rules are grouped with the operation they support.
- Testing can focus on inputs and outputs.

Classes may still exist where they provide a strong boundary—such as a file parser, database adapter, HTTP client, or UI component. The difference is that objects are used deliberately rather than treated as the universal unit of software design.

## 8. The practical lesson: prefer clarity over ideology

“OOP is bad” is deliberately provocative. The more practical conclusion is that OOP becomes harmful when its rules override the shape of the problem.

Use objects when they provide a clear abstraction with a small, coherent responsibility. Be suspicious when an object becomes a miniature application with dozens of collaborators, when inheritance models a changing business domain, or when a simple workflow requires navigating through a maze of classes.

Good software is not defined by whether it is object-oriented, functional, procedural, or data-oriented. It is defined by whether a developer can answer basic questions quickly:

- Where does this behavior live?
- What data does it use?
- What can it change?
- What depends on it?
- How can I test it?

If OOP makes those answers harder to find, then it is not helping—it is getting in the way.
