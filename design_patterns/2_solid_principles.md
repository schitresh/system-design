## SOLID Principles
- The SOLID principle help in reducing tight coupling
  - Tight coupling means a group of classes are highly dependent on one another
- Our code should be loosely coupled
  - Loosely coupled classes minimize changes in the code
  - Helps in making code more reusable, maintainable, flexible and stable

## Single Responsibility
- A class should have only one responsibility
  - In other words, only one reason to change
- Importance
  - Less coupling and fewer dependencies
  - Well organized classes, easier to understand and modify
  - Better testing due to fewer test cases
- Example
  - Imagine a baker who is responsible for baking bread
    - Ensuring that the bread is of high quality and properly baked
  - If the baker is also responsible for other tasks, it will violate single responsibilty
    - Like managing the inventory, ordering supplies, serving customers, etc.

## Open for Extension, Closed for Modification
- Software entities (classes, modules, functions, etc.)
  - Should be open for extension, but closed for modification
  - We should be able to extend a class behavior without modifying it
- Importance
  - New features can be added without modifying existing code
  - Adapts to changing requirements more easily
  - Saves testing the existing implementations and avoids potential bugs
- Example
  - Imagine we have a PaymentProcessor class to processes payments for an online store
    - Initially, it only supported processing payments using credit cards
    - Now, we want to extend its functionality to also support PayPal
  - Instead of modifying the PaymentProcessor class to add PayPal support
    - Create a new class PaypalPaymentProcessor that extends the PaymentProcessor class

## Liskov Substitution
- Functions that use pointers or references to base classes
  - Must be able to use objects of derived classes without knowing it
- If class B is a subtype (or child) of class A
  - We should be able to replace A with B
    - Without disrupting the behavior of program
- Importance
  - Enables the use of polymorphic behavior, making code more flexible and reusable
  - Ensures that subclasses adhere to the contract defined by the superclass
  - Guarantees that replacing a superclass object with a subclass object won't break the program
- Example
  - A rectangle’s can have any height and width
  - A square is a rectangle with equal width and height
  - So the properties of the rectangle class should be extendable into square class

## Interface Segregation
- Clients should not be forced to depend upon interfaces that they do not use
  - Prefer many client interfaces rather than one general interface
    - Larger interfaces should be split into smaller ones
    - Each interface should have a specific responsibility
  - The implementing class should only be concerned about the methods that are connected
- Importance
  - Reduces dependencies between classes, making the code more modular and maintainable
  - Allows for more targeted implementations of interfaces
  - Avoids unnecessary dependencies, clients don't have to depend on methods they don't use
- Example
  - Suppose we enter a restaurant and we are pure vegetarian
  - But the menu card includes vegetarian, non-vegetarian, drinks, sweets, etc.
  - The menu should be different for different types of customers

## Dependency Inversion
- Instead of high level modules depending on low level modules
  - Both should depend on abstractions
- Additionally, abstractions should not depend on details, details should depend on abstractions
  - Classes should rely on abstractions (e.g., interfaces or abstract classes)
    - Rather than concrete implementations
- Importance
  - Reduces dependencies between modules, making the code more flexible and easier to test
  - Enables changes to implementations without affecting clients
  - Makes code easier to understand and modify
- Example
  - Developers depend on an abstract version control system like git
  - They don’t depend on specific details of how git works internally
