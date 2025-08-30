# Strategy

-   Defines a family of algorithms
    -   Puts each of them into a separate class
    -   Makes their objects interchangeable

## Problem

-   Let's say we are creating a navigation app for casual travelers
    -   The app is centered around a beautiful map to help users quickly orient themselves in any city
-   It also has automatic route planning
    -   A user can enter an address and see the fastest route to the destination on the map
    -   In the first version, we could only build the driving routes over roads
    -   Later, we also added walking routes and public transport routes
-   We also have plans to add routes for cyclists
    -   And routes through all of a city's tourist attractions
-   But there are many technical difficulties
    -   Each time a new routing algorithm is added, the main class of the navigator doubles in size
    -   Any change to one of the algorithm affects the whole class
        -   Even a simple bug fix or a slight adjustment to street score

## Solution

-   Take a class that does something specific in a lot of different ways
    -   And extract all of these algorithms into separate classes called strategies
    -   The original class (called context) must store a reference to one of the strategies
    -   The context should delegate the work to the linked strategy
-   The context isn't responsible for selecting an appropriate algorithm
    -   The client passes the desired strategy to the context
    -   The context works with all the strategies through a generic interface
    -   The interface exposes only a single method to trigger the algorithm
        -   Encapsulated within the selected strategy
-   This makes the context and the strategies independent of each other
    -   Can add new algorithms or modify existing ones without changing the context or other strategies
-   In the navigation app, each routing algorithm can be extracted to its own class
    -   Which will return a collection of the route’s checkpoints
    -   The main navigator can switch the active routing strategy
-   This may look similar to the State pattern
    -   But in this state pattern, states are aware of each other and initiate transitions
    -   While in strategy pattern, strategies almost never know about each other

## Example

```rb
# Context
# Maintains a reference to the currently choosen strategy
class PaymentSystem
  attr_accessor :payment_mode

  def initialize(amount, payment_mode)
    @amount = amount
    @payment_mode = payment_mode
  end

  def pay
    @payment_mode.pay(@amount)
  end
end

# Strategy
class PaymentMode
  def pay(amount)
    raise('Not Implemented')
  end
end

# Concrete Strategy
class Cash < Payment
  def pay(amount)
    puts "You paid $#{amount} by cash"
  end
end

class CreditCard < Payment
  def pay(amount)
    puts "You paid $#{amount} by credit card"
  end
end

# Client Code
def pay(mode)
  cash = Cash.new
  credit_card = CreditCard.new

  payment_system = PaymentSystem.new(100, credit_card)
  payment_system.pay
  return if payment_system.paid?

  # Let's say the above pay failed, client changes the mode to cach
  payment_system.payment_mode = cash
  payment_system.pay
end
```
