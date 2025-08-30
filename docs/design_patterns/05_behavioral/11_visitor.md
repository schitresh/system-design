# Visitor

-   Separates algorithms from the objects on which they operate

## Problem

-   Let's say we are developing an app which works with geographic information
    -   Structured as one colossal graph
    -   Each node of the graph represents a complex entity (like city)
        -   Or a granular entity (like industry, sightseeing area)
    -   The nodes are connected with others if there's a road between them
    -   Each node type is represented by its own class and each specific node is an object
-   Let's say we want to allow exporting the graph into XML format
    -   The basic approach is to add an export method to each node class
    -   And leverage recursion to go over each node
-   But it might not make sense to have the XML export code within the node classes
    -   Because the primary job of these classes is to work with geodata
    -   Also, we don't want to risk breaking the core functionality
        -   Because of potential bugs in the export code
    -   In future, we may also get a request to export in different formats
        -   Or some other related features

## Solution

-   Place the new behavior into a separate class called visitor
    -   Instead of intergrating it into existing classes
-   But the actual implementation may differ across various node classes
    -   City node & industry node might export their data differently
    -   Maybe, the visitor class can take arguments
        -   And define methods to handle different requirements
-   But how exactly would we call these methods
    -   Especially when dealing with the whole graph
    -   These methods have different signatures, so we can’t use polymorphism
    -   To pick a proper visitor method that’s able to process a given object
        -   We’d need to check its class
-   Use double dispatch method
    -   Instead of letting the client select a proper version of the method to call
    -   Delegate this choice to the objects we're passing to the visitor
        -   Since the objects know their own class, they can pick a proper method
    -   This adds some changes in the node classes but they are trivial

## Example

```rb
# Acceptors
class Motherboard
  attr_accessor :brand, :model

  def initialize(brand, model)
    @brand = brand
    @model = model
  end

  def accept(visitor)
    visitor.visit(self)
  end
end

class Cpu
  attr_accessor :brand, :model

  def initialize(brand, model)
    @brand = brand
    @model = model
  end

  def accept(visitor)
    visitor.visit(self)
  end
end

class Computer
  def initialize(motherboard, cpu, dram, hdd)
    @parts = [motherboard, cpu]
  end

  def accept(visitor)
    visitor.visit_parts(@parts)
  end
end

# Visitors
class ComputerBrand
  def visit(acceptor)
    puts "Brand: #{acceptor.brand}"
  end

  def visit_parts(parts)
    parts.each do |part|
      part.accept(visitor)
    end
  end
end

class ComputerModel
  def visit(acceptor)
    puts "Model: #{acceptor.model}"
  end

  def visit_parts(parts)
    parts.each do |part|
      part.accept(visitor)
    end
  end
end

# Client
def computer_brand_and_model
  motherboard = Motherboard.new("ASUS", "X99-DELUXE")
  cpu = Cpu.new("Intel", "Core i7-5960X")
  computer = Computer.new(motherboard, cpu)

  computer.accept(ComputerBrand.new)
  # Brand: ASUS
  # Brand: Intel

  computer.accept(ComputerModel.new)
  # Model: X99-DELUXE
  # Model: Core i7-5960X
end
```
