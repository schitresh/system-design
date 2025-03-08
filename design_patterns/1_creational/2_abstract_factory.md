## Abstract Factory
- Allows to produce families of related objects without specifying their concrete classes
- Products generated form a factory are compatible to each other
- Decoupling between concrete products and client code

## Problem
- Let's say we want to sell furnitures like chair, sofa, etc.
- And these are available in two variants: modern, traditional
- Variants of the individual furnitures should match with each other
- If a customer orders modern furniture, all the items (chair, sofa) should be modern

## Solution
- Explicitly declare interfaces for each distinct product (chair, sofa)
  - Make all the variants of the product follow these interfaces
  - So that all the chair variants will implement the Chair interface
- After that declare the abstract factory
  - Interface with the list of creation methods of all products
  - For each variant of a product, create a factory class based on the abstract factory
- If a client wants a factory to produce a chair
  - It doesn't have to be aware of the factory's class
  - It should treat all the variants of the chair in the same mannerr
  - It should match the variant of other products like sofa produced by the same factory object

## Example
```rb
# Abstract Factory
# Interface with separate methods for the products
class Furniture
  def build_chair
    raise('Not Implemented')
  end

  def build_sofa
    raise('Not Implemented')
  end
end

# Concrete Factory
# Implements the abstract factory for a single variant for the family of products
class ModernFurniture < Furniture
  def build_chair
  end

  def build_sofa
  end
end

class TraditionalFurniture < Furniture
  def build_chair
  end

  def build_sofa
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

class ModernChair < Chair
end

class TraditionalChair < Chair
end

# Client
def build
  furniture = ModernFurniture.new
  furniture.build_chair
end
```
