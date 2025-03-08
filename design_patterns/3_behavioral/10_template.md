## Template
- Defines the skeleton of an algorithm but lets subclassees override specific steps of the algorithm without changing its structure

## Problem
- Let's say we're creating a data mining application that analyzes corporate documents
- Users feed documents in various formats (pdf, doc, csv)
- It tries to extract meaningful data from these doccuments in a uniform format
- You observe that the the date miner classes for these formats have a lot of similar code
  - For example, the code for data processing and analysis is similar
- Also, these different classes introduced conditionals in the client code
  - If there was a commoon interface, this could have been eliminated

## Solution
- Break down the algorithms into a series of steps and turn these steps into methods
- Put a series of calls to thesee methods inside a single template method
- The steps may either be abstract or haveg some default implementation
- The subclasses will implement the abstract steps and override some of the optional one sif required
- We can also provide hooks as optional steps
  - Hooks are placed before or after the crucial steps of an algorithm
  - This provides subclasses with additional extenstion points for an algorithm

## Example
```rb
# Abstract Class
# Defines a template method composed of calls to abstract primitive operations
class Sandwich
  def initialize(bread, condiment)
    @bread = bread
    @condiment = condiment
  end

  def make
    arrange_bread
    spread_condiment

    before_vegetable_hook

    put_vegetable
    put_filling

    after_filling_hook
  end

  # Optional operation which can be overrided by subclasses
  def arrange_bread
    puts "Arranged #{@bread}"
  end

  def spread_condiment
    puts "Spreaded #{@condiment}"
  end

  # Required Operation
  def put_vegetable
    raise('Not Implemented')
  end

  def put_filling
    put_base_filling
  end
end

# Concrete Classes
class CheeseSandwich < Sandwich
  def initialize(bread, condiment)
    super
  end

  def before_vegetable_hook
    put_cheese
  end

  def put_vegetable
    put_base_vegetables
  end
end

class VegSandwich < Sandwich
  def initialize(bread, condiment, vegetables)
    super(bread, condiment)
    @vegetables = vegetables
  end

  def put_vegetable
    put_base_vegetables
    put_input_vegetables
  end
end

# Client
def make_sandwiches
  cheese_sandwich = CheeseSandwich.new('Multigrain Bread', 'Ketchup')
  cheese_sandwich.make

  veg_sandwich = VegSandwich.new('Oregano Bread', 'Butter', ['Lettuce'])
  veg_sandwich.make
end
```
