# Abstract Factory

-   Allows to produce families of related objects without specifying their concrete classes
-   Products generated form a factory are compatible to each other
-   Decoupling between concrete products and client code

## Problem

-   Let's say we want to sell furnitures like chair, sofa, etc.
    -   The furnitures are available in two variants: modern, traditional
-   We need a way to create individual furniture objects
    -   So that they match other objects of the same family
    -   If a customer orders modern furniture, all the items like chair & sofa should be modern
-   Furniture vendors update their catalogs very often
    -   So we wouldn’t want to change the core code when adding new items or variants

## Solution

-   Explicitly declare interfaces for each distinct product (chair, sofa)
    -   Then make all the variants of the product follow these interfaces
    -   So, all the chair variants will implement the Chair interface
-   After that, declare the abstract factory
    -   An interface with a list of creation methods of all the products
    -   For each variant of a product, create a factory class based on the abstract factory
-   The client code works with both factories and products via their abstract interfaces
    -   This lets us change the type of a factory passed to the client code
        -   And the product variant that the client code receives
    -   The client shouldn’t care about the factory class it works with
-   If a client wants a factory to produce a chair
    -   It doesn't have to be aware of the factory's class
    -   It should treat all the variants of the chair in the same manner
    -   It should match the variant of other products like sofa produced by the same factory object

## Example

```rb
# Abstract Factory
# Interface that declares separate methods to return different abstract products.
# These products are called a family and are related by a high-level theme or concept.
# Products of one family are usually able to collaborate among themselves.
# A family of products may have several variants, but the products of one variant are
# incompatible with products of another.
class Furniture
  def build_chair
    raise('Not Implemented')
  end

  def build_sofa
    raise('Not Implemented')
  end
end

# Concrete Factory
# Produces a family of products that belong to a single variant
class ModernFurniture < Furniture
  def build_chair
    ModernChair.new
  end

  def build_sofa
    ModernSofa.new
  end
end

class TraditionalFurniture < Furniture
  def build_chair
    TraditionalChair.new
  end

  def build_sofa
    TraditionalSofa.new
  end
end

# Abstract Product
class Chair
end

# Concrete Product
class ModernChair < Chair
end

class TraditionalChair < Chair
end

# Abstract Product
class Sofa
end

class ModernSofa < Sofa
end

class TraditionalSofa < Sofa
end

# Client Code
def build_furniture(factory)
  factory.build_chair
  factory.build_sofa
end

factory = ModernFurniture.new
build_furniture(factory)
```
