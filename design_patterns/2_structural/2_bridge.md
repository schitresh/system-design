## Bridge
- Splits a large class or set of closely related classes into two seperate hierarchies (abstraction & implementation)
- So that they can be developed independently of each other

## Problem
- Let's say we have a geometric Shape class with a pair of subclasses Circle & Square
- We want to extend this class heirarchy to incorporate colors Red & Blue
- So we'll need to create four class combinations RedCircle, RedSquare, BlueCircle, BlueSquare
- Adding new shapes or new colors will increase the hierarchy exponentially

## Solution
- Switch from inheritance to object composition
- Extract one of the dimensions into a separate class hierarchy
  - So that the original classes will reference an object of the new heirarchy
  - Instead of having all of its state & behaviors within one class
- There will be Color class with Red & Blue subclasses, which will be referenced by the Shape class

### Abstraction & Implementation
- Abstraction (or interface) is a high level control layer for some entities
- It doesn't do any real work and delegates it to the implementation layer (or platform)
- For example,
  - Abstraction can be represented by a GUI
    - Different GUIs for different types of users (customers or admins)
  - Implementation can be the underlying operating system code (API) which the GUI calls
    - Different APIs for different platforms (windows, linux, macOS)


## Example
```rb
# Abstraction
class Shape
  attr_reader :color

  def initialize(color)
    # Reference to an object of the 'implementation' heirarchy
    @color = color
  end

  def draw
    raise('Not Implemented')
  end
end

# Extended Abstraction
class Square < Shape
  def draw
    pen = color.load_pen
    pen.draw_line
  end
end

class Circle < Shape
end

# Implementation
class Color
end

# Concrete Implementation
class Red < Color
end

class Blue < Color
end

# Client
def draw_shape
  color = Red.new
  shape = Square.new(color)
  shape.draw
end
```
