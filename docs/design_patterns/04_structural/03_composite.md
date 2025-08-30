# Composite

-   Also known as Object Tree
-   Composes objects into tree structures
    -   And works with these structures as if they were individual objects
-   Used only when the core model of an app can be represented as a tree

## Problem

-   Let's say we want to create an ordering system that uses Products & Boxes
    -   A box can contain several products as well as other smaller boxes
    -   The smaller boxes can also hold some products or even smaller boxes and so on
-   To determine the total price of an order
    -   The direct approach will be to unwrap all the boxes and go over all the products
    -   But the nesting level and other nasty details can make it complicated

## Solution

-   Create a common interface that can calculate the total price
    -   For a product, it will simply consider the product's price
    -   For a box, it will go over each item and ask its price
-   The smaller boxes can individually go over their contents and get the price
    -   This way we don't need to worry about how the boxes calculate their price
    -   And a box can also add some extra cost like packaging cost
-   We don’t need to care about the concrete classes of objects
    -   The objects themselves pass the request down the tree
    -   If a box is simple or sophisticated or gift wrapped
        -   It can handle these criterias on its own

## Example

```rb
# Component (Common Interface)
# Declares common operations for both simple and complex objects of a composition
class Component
  attr_accessor :parent

  def add(component)
    raise('Not Implemented')
  end

  def remove(component)
    raise('Not Implemented')
  end

  def composite?
    false
  end

  def price
    raise('Not Implemented')
  end
end

# Leaf
# End objects of a composition, can't have any children.
# Usually, it's the leaf objects that do the actual work, whereas Composite
# objects only delegate to their sub-components.
class Product < Component
end

# Composite
# Complex components that may have children
# Usually delegates the actual work to the children and sum up the results
class Box < Component
  def initialize
    @children = []
  end

  def add(component)
    @children.append(component)
    component.parent = self
  end

  def remove(component)
    @children.remove(component)
    component.parent = nil
  end

  def composite?
    true
  end

  def price
  end
end

# Client Code
def get_price
  product1 = Product.new
  box1 = Box.new
  box1.add(product1)

  product2 = Product.new
  box2 = Box.new
  box2.add(product2)

  box = Box.new
  box.add(box1)
  box.add(box2)
  box.add(product3)
  box.price
end
```
