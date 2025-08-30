# Design Patterns

-   Blueprint to commonly occuring problems in software design that can be customized
-   The patterns are general concepts that can be applied to solve specific problems
-   The code of the same pattern applied to two different programs may be different

## Types of Design Patterns

-   Creational: Object creation mechanisms that provide flexibility and reusability
-   Structural: How to assemble objects and classes into larger structures
    -   And keeping these structures flexible and efficient
-   Behavioral: Concerned with algorithms and assignment of responsibility among objects

## Creational

-   Factory: Create objects of derived classes
-   Abstract Factory: Create objects of families of classes
-   Builder: Complex object construction with different representations
-   Prototype: Clone a fully initialized object
-   Singleton: Class for which only a single object exists

## Structural

-   Adapter: Collaboration of objects with incompatible interfaces
-   Bridge: Split a large class or related classes into two separate independent hierarchies
    -   Abstraction & implementation
-   Composite: Compose objects into tree structures as if they were individual objects
-   Decorator: Attach new behaviors to objects using wrapper objects
-   Facade: Simplified interface to a library, framework or a set of classes
-   Flyweight: Share common states among objects instead of keeping all the data in each object
-   Proxy: Placeholder for original object
    -   Control access to the original object
    -   Perform pre or post request tasks

## Behavioral

-   Chain of Responsibility: Pass requests along a chain of handlers
    -   That decide to process or pass to the next handler
-   Command: Encapsulate a command request as an object
-   Interpreter: Include language elements in a program
-   Iterator: Traverse elements of a collection without exposing its underlying representation
-   Mediator: Restrict direct communication and reduce chaotic dependencies between the objects
    -   Via a mediator object
-   Memento: Capture and restore the previous internal state of objects
-   Observer: Subscription to notify objects about events of the object under observation
-   State: Alter an object's behavior when its state changes as if the object changed its class
-   Strategy: Encapsulate a family of algorithms inside classes with interchangeable objects
-   Template: Defer the exact steps of an algorithm to a subclass without changing its structure
-   Visitor: Defines a new operation to a class without change
