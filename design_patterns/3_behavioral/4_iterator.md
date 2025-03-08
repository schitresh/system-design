## Iterator
- Traverse elements of a collection without exposing its underlying representation (list, stack, tree, etc.)

## Problem
- How to go through each element of a collection without accessing the same elements over and over
- Especially for complex data structures like trees
- There can be some algorithms custom tailored for a specific application

## Solution
- Encapsulate all the traversal details in an iterator object like current position, elements left till the end
- All iterators must implement the same interface
  - To make the code compatible with any collection type or any traversal algorithm

## Example
```rb
# External Iterator
class ArrayIterator
  attr_reader :array
  attr_accessor :index

  def initialize(array)
    @array = array
    @index = 0
  end

  def has_next?
    index < array.length
  end

  def item
    array[item]
  end

  def next_item
    value = item
    index += 1
    value
  end
end

# Internal Iterator
class Account
  attr_accessor :name, :balance

  def initialize(name, balance)
    @name = name
    @balance = balance
  end

  def <=>(other)
    balance <=> other.balance
  end
end

class Portfolio
  include Enumerable
  attr_accessor :accounts

  def initialize
    @accounts = []
  end

  def each(&block)
    accounts.each(&block)
  end

  def add_account(account)
    accounts << account
  end
end

# Client
def check_portfolio
  portfolio = Portfolio.new
  portfolio.add_account(Account.new('Bonds', 200))
  portfolio.add_account(Account.new('Stocks', 100))

  portfolio.all? { |account| account.balance >= 10 }
  portfolio.map { |account| account.balance }
end
```
