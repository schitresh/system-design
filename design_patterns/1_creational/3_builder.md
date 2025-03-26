## Builder
- Constructs complex objects
  - That require step by step initialization of many fields and nested objects
- Allows to produce different types and representations of an object
  - Using the same construction code

## Problem
- Imagine a complex object
  - That requires step-by-step initialization of many fields and nested objects
  - Such initialization is usually inside a large constructor with lots of parameters
  - It can become too complex by creating subclasses for every possible configuration
- Let's say we want to create a House object
  - To build a simple house, we need to construct walls, floor, door, windows, roof
  - After that we will also need to add plumbling, electrical wiring, etc.
  - The client may also want backyard, heating system, air conditioning
- The simplest solution is to extend the base House class and create subclasses
  - But eventually we'll end up with a lot of subclasses to cover all the combinations
  - Any new parameters like porch style will keep growing this heirarchy
- Another solution can be to create a large constructor in the base House class
  - But in most cases, many of the parameters will be unused
  - Because only a fraction of house will have swimming pools
  - This will make the constructor calls ugly

## Solution
- Extract the object construction out of its own class
  - And move it to separate objects called builders
- Organize object construction into a set of steps
  - That can be executed on a builder object
- Some steps might require different implementation to build various representations
  - Walls of a cabin must be built of wood
  - Walls of a castle walls must be built with stone
- Create several different builder classes
  - That implement the same set of building steps but in a different manner
  - Call only those steps that are necessary for a particular configuration

### Director
- We can go further and extract a series of calls to a director class
- Director class defines the order to execute the building steps
  - While the builder provides the implementation for those steps
- It extracts the details of product construction from the client code
  - The client only needs to associate a builder with a director

## Example
```rb
# Builder
# Declares product construction steps common to all types of builders
class HouseBuilder
  def build_walls
    raise('Not Implemented')
  end

  def build_floor
    raise('Not Implemented')
  end
end

# Concrete Builder
# Provides different implementations for the construction steps
def CabinBuilder < HouseBuilder
  def build_walls
    cut_wood_logs
  end
end

def CastleBuilder < HouseBuilder
  def build_walls
    break_rocks_into_stones
  end
end

# Product
# The resulting objects contructed by different builders
def Cabin
end

def Castle
end

# Director
def Director
  attr_reader :builder

  def initialize(builder)
    @builder = builder
  end

  def build_room
    builder.build_walls
    builder.build_floor
  end

  def build_swimming_pool
  end
end

# Client Code
def build
  builder = CabinBuilder.new
  director = Director.new(builder)
  director.build_room
end
```
