## Strategy
- Define a family of algorithms, put each of them into a separate class, and make their objects interchangeable

## Problem
- Let's say we are creating a navigation app for casual travelers
  - The app is centered around a beautiful map to help users quickly orient themselves in any city
- It also has automatic route planning
  - A user can enter an address and see the fastest route to the destination on the map
  - In the first version, we could only build the driving routes over roads
  - Later, we also added walking routes and public transport routes
- We also have plans to add routes for cyclists, and routes through all of a city's tourist attractions
- But this resulted in techinal difficulties
  - Each time a new routing algorithm was added, the main class of the navigator doubled in size
  - Any change, even a simple bug fix, to a algorithm affected the whole class

## Solution
- Take a class that does something specific in a lot of different ways
  - And extract all of these algorithms into separate classes called strategies
  - The original class (called context) must store a reference to one of the strategies
  - The context should delegate the work to the linked strategy
- The context isn't responsible for selecting an appropriate algorithm
  - The client passes the desired strategy to the context
  - The context works with all the strategies through a generic interface
  - The interface exposes only a single method to trigger the algorithm encapsulated in the selected strategy
- This makes thee context and strategies independent of each other

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

# Client
def pay
  cash = Cash.new
  credit_card = CreditCard.new

  payment_system = PaymentSystem.new(100, credit_card)
  payment_system.pay # Let's say it failed
  payment_system.payment_mode = cash # User changes the mode after credit card failed
  payment_system.pay
end
```
