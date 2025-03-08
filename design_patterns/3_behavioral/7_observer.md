## Observer
- Define a subscription mechanism to notify multiple objects about any events that happen to the object they're observing

## Problem
- Let's say we have two types of objects, Customer and Store
- If a customer is interested in a particular brand or product (say a new model of iphone)
- And the customer wants to know whenever it becomes available in a store
- If the customer visits the store every day, most of the trips will be pointless
- If the store sends emails to all customers about all the new products, it will be upset the customers

## Solution
- Add a subscription mechanism to the publisher class
- Individual objects can subscribe to or unsubscribe from a stream of events coming from that publisher
- Whenever an event happens, the publisher goes over its subscribers
  - And calls the specific notification method on their objects
- There can be a lot of subscriber classes, so the publisher shouldn't be coupled with them
  - We might not even know about some of them beforehand
  - So all subscribers should implement the same interface
  - The publisher should communicate with them only via that interface
- This interface should declare the notification method
  - Along with a set of paramters that the publisher can use to pass some contextual data


## Example
```rb
module Observers
  def subscribe(observer)
    @observers << observer
  end

  def unsubscribe(observer)
    @observers.delete(observer)
  end

  def publish
    @observers.each do |observer|
      observer.process_event(self)
    end
  end
end

class Employee
  include Observers

  attr_reader :name, :title, :salary

  def initialize(name, title, salary)
    @name = name
    @title = title
    @salary = salary
    @observers = []
  end

  def salary=(new_salary)
    @salary = new_salary
    publish
  end
end

class Payroll
  def process_event(data)
  end
end

# Client
def subscribe_to_employee_events
  employee = Employee.new('Name', 'Title', 100000)

  payroll = Payroll.new
  employee.subscribe(payroll)

  tax_tracker = TaxTracker.new
  employee.subscribe(tax_tracker)

  employee.salary = 500000
end
```
