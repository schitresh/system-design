# Chain of Responsibility

-   Also known as Chain of Command
-   Passes requests along a chain of handlers
-   Upon receiving a request, each handler decides to either process the request
    -   Or pass it to next handler in the chain

## Problem

-   Let's say we're working on an online ordering system
    -   We want to restrict access to the system so that only authenticated users can create orders
    -   User with administrative permissions must have full access to all orders
-   After some planning, we realized that these checks must be performed sequentially
    -   Sanitize the data in a request because it's unsafe to pass raw data straight to the ordering system
    -   Filter repeated failed request from the same IP to restrict brute force password cracking
    -   Speed up the system by returning cached results on repeated requests with the same data
-   This leads to a complex checking system and changing one check affects the others
-   Also, it can lead to duplicate code if a component require some of these checks but not all

## Solution

-   Transform particular behaviors into stand-alone objects called handlers
-   Each check should be extracted to its own class with a single method that performs the check
-   Each linked handler has a field for storing a reference to the next handler in the chain
-   The request travels along the chain until all handlers process it
-   A handler can decide not to pass the request further down and effectively stop further processing

## Example

```rb
# Handler
class Policy
  def next_policy=(policy)
    raise('Not Implemented')
  end

  def validate(order)
    raise('Not Implemented')
  end
end

# Abstract Handler
class OrderPolicy < Policy
  def next_policy=(policy)
    @next_policy = policy
    policy
  end

  def validate(order)
    return true if @next_policy.blank?

    @next_policy.validate(order)
  end
end

# Concrete Handler
class WeightPolicy < OrderPolicy
  def validate(order)
    if order.weight <= order.type.weight_limit
      # Execute next policy
      super(order)
    else
      false
    end
  end
end

class PaymentPolicy < OrderPolicy
  def validate(order)
    if order.payment_details
      # Execute next policy
      super(order)
    else
      false
    end
  end
end

# Client Code
def validate_orders(policy)
  weight_policy = WeightPolicy.new
  payment_policy = PaymentPolicy.new
  weight_policy.next_policy = payment_policy

  orders.each do |order|
    weight_policy.validate(order)
  end
end
```
