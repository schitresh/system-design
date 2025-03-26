## Factory
- Also known as Virtual Constructor
- Provides an interface for creating objects in a superclass
  - But allows subclasses to alter the type of objects that will be created
- Allows creating product objects without specifying their concrete classes
  - Avoids tight coupling between the creators and the products

## Problem
- Let's say we're creating a logistics management application
  - The app handles transportation by trucks
  - So the bulk of the code is in the Truck class
- After a while, we receive requests to incorporate sea logistics into the app
  - Adding a new Ship would require a lot of changes
  - Since the the code is coupled to the Truck class
- Adding yet another type of transportation will require making these changes again
  - This will introduce many conditionals to switch the app's behavior
    - Depending on the class of the transportation objects

## Solution
- Replace the direct object construction calls with calls to a special factory method
  - Objects returned by a factory method are often referred as products
- This allows us to override the factory method in a subclass
  - And change the class of products being created by the method
  - But these products from all the subclasses should have a common base class or interface
- The Truck class and the Ship class implement the Transport interface
  - The Transport interface declares a method called deliver
  - Truck implements it to deliver by land, Ship implements it to deliver by sea
- The code that uses the factory method
  - Doesn’t see a difference between the products returned by various subclasses
  - It knows that all transport objects are supposed to have the deliver method
    - But exactly how it works isn’t important to it

## Example
```rb
# Creator
# Declares the factory method that returns an object of a product class. The subclasses
# provide the implementation, but it may provide a default implementation.
# Its primary responsibility isn't creating products, it usually contains some core
# business logic that relies on product objects returned by the factory method.
# Subclasses can indirectly change that business logic by overriding the factory method
# and returning a different type of product.
class Transport
  def deliver
    raise('Not Implemented Error')
  end
end

# Concrete Creator
# Implements the creator interface
class Truck < Transport
  def deliver(parcel_details)
    Box.new(parcel_details)
  end
end

class Ship < Transport
  def deliver(parcel_details)
    Container.new(parcel_details)
  end
end

# Product
# Declares the interface common for all objects that are produced by the creators
class Parcel
  def pick
    raise('Not Implemented Error')
  end
end

# Concrete Product
# Implements the product interface
class Box < Parcel
  def pick
  end
end

class Container < Parcel
  def pick
  end
end

# Client Code
def pick_parcel(transporter, parcel_details)
  parcel = transporter.deliver(parcel_details)
  parcel.pick
end

parcel_details = { height: '1m', width: '1m', weight: '10kg' }
transporter = Truck.new
pick_parcel(transporter, parcel_details)
```
