## Single Responsibility
- A class should have single responsibility
  - In other words, only one reason to change
- Advantages
  - Less coupling and fewer dependencies
  - Well organized classes
  - Better testing due to fewer test cases

## Open for Extension, Closed for Modification
- We should be able to extend a class behavior without modifying it
- This saves testing the existing implementations and avoids potential bugs

## Liskov Substitution
- If class A is a subtype of class B
  - We should be able to replace B with A without disrupting the behavior of program

## Interface Segregation
- Larger interfaces should be split into smaller ones
- Implementing class only needs to be concerned about the methods that are connected

## Dependency Inversion
- Instead of high level modules depending on low level modules
  - Both should depend on abstractions
