## Factory
- Provides an interface for creating objects
- Allows subclasses to alter the type of objects that will be created
- Avoids tight coupling between the creators and the products

## Problem
- Let's say we're creating a logistics management application
- Currently the transportation is handled by trucks, so the code uses Truck class everywhere
- If we start sea transportation later, adding Ship class would require a lot of changes
- The same thing will happen if we add more modes of transport

## Solution
- Replace direct object contruction calls with calls to a special factory method
- Objects returned by a factory method are referred as products
- This factory method can be overriden in a subclass to change the class of products
- The code that uses the factory method doesn't see a differencee between the products returned by various subclasses
- Both Truck and Ship classes should implement Vehicle interface

## Example
```rb
# Creator
# Declares the factory method that returns new product objects
# It can return a default product type of leave it to the subclasses
# Product creation is not the primary responsibility of the creator
# It already has some core business logic related to the products
class Vehicle
  def pick_parcel
    raise('Not Implemented')
  end
end

# Concrete Creator
# Implementation of the creator interface
class Truck < Vehicle
  def pick_parcel(parcel_details)
    parcel = Box.new(parcel_details)
  end
end

class Ship < Vehicle
  def pick_parcel(parcel_details)
    parcel = Container.new(parcel_details)
  end
end

# Product
# Declares the interface common to all objects that are produced by the creators
class Parcel
  def pick
    raise('Not Implemented')
  end
end

# Concrete Product
# Implementations of the product interface
class Box < Parcel
  def pick
  end
end

class Container < Parcel
  def pick
  end
end

# Client
def pick_parcel
  vehicle = Truck.new
  parcel_details = { height: '1m', width: '1m', weight: '10kg' }
  vehicle.pick_parcel(parcel_details)
end
```
