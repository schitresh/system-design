# Iterator

-   Traverses elements of a collection without exposing its underlying representation (list, stack, tree, etc.)

## Problem

-   How to go sequentially traverse each element of a collection
    -   Without accessing the same elements over and over
    -   Especially for complex data structures like trees, graphs
-   Also we might need different types of traversals
    -   Like DFS, BFS, random access for tree elements
    -   Adding more and more traversal algorithms to the collection gradually blurs
        -   Its primary responsibility which is efficient data storage
    -   Some algorithms might be tailored for a specific application
        -   Including them into a generic collection class may not be a good idea
-   The client code may not care how a collection stores elements
    -   But since collections provide different ways of accessing their elements
    -   We have no option other than to couple the code to the specific collection

## Solution

-   Extract the traversal behavior of a collection into a separate object called iterator
    -   It implements the traversal algorithm
    -   Also encapsulates all of the traversal details such as
        -   The current position, how many elements are left till the end
    -   This allows several iterators to through the same collection at the same time independently
-   All iterators must implement the same interface
    -   This makes the client code compatible with any collection type or any traversal algorithm
    -   If we need a special way to traverse a collection, just create a new iterator class
        -   Without having to change the collection or the client

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

# Client Code
def check_portfolio
  portfolio = Portfolio.new
  portfolio.add_account(Account.new('Bonds', 200))
  portfolio.add_account(Account.new('Stocks', 100))

  portfolio.all? { |account| account.balance >= 10 }
  portfolio.map { |account| account.balance }
end
```
