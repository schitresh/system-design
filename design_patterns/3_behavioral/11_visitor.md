## Visitor
- Separates algorithms from the objects on which they operate

## Problem
- Let's say we are developing an app which works with geographic information structured as a colossal graph
- Each node of the graph represents a complex entity (like city) or a granular entity (like industry, sightseeing area)
  - The nodes are connected with others if there's a road between them
  - Each node type is represented by its own class and each specific node is an object
- Let's say we want to allow exporting the graph into XML format
  - The basic approach is to add an export method to each node class and leverage recursion to go over each node
- But it might not make sense to have the XML export code within the node classes
  - Because the primary job of these classes is to work with geodata
  - And we don't want to risk breaking the core functionality because of potential bugs in the export code
  - Also, what if we get a request to export in different formats or some other related logic

## Solution
- Place the new behavior into a separate class called visitor instead of intergrating it into existing classes
- What if the actual implementation differs across various node classes
  - City node & industry node might export their data differently
- Visitor class can take arguments and define methods to handle different requirements
  - But that will require many conditionals or checking the node's class
- Use double dispatch method
  - Instead of lettig the client select a proper version of the method to call
  - Delegate this choice to the objects we're passing to the visitor
  - This adds some changes in the node classes but they are trivial

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
