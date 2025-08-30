# Bridge

-   Splits a large class or a set of closely related classes
    -   Into two seperate hierarchies - abstraction & implementation
    -   So that they can be developed independently of each other

## Problem

-   Let's say we have a geometric Shape class with a pair of subclasses Circle & Square
-   We want to extend this class heirarchy to incorporate colors Red & Blue
    -   So we'll need to create four class combinations
        -   RedCircle, RedSquare, BlueCircle, BlueSquare
-   Adding new shapes or new colors will increase the hierarchy exponentially
    -   To add Triangle shape, we would need one subclass for each color
    -   Adding a new color would require creating one subclass for each shape

## Solution

-   This problem occurs because we’re trying to extend the shape classes in
    -   Two independent dimension - by form and by color
    -   That’s a very common issue with class inheritance
-   Switching from inheritance to the object composition can solve this problem
    -   Extract one of the dimensions into a separate class hierarchy
    -   So that the original classes will reference an object of the new hierarchy
    -   Instead of having all of its state and behaviors within one class
-   Create a Color class with two subclasses - Red and Blue
    -   The Shape class then gets a reference field pointing to one of the color objects
    -   The reference will act as a bridge between the Shape and Color classes
    -   Any color related logic can be delegated to the linked color object

### Abstraction & Implementation

-   This is different from the interfaces or abstract classes of a programming language
-   Abstraction (also called interface) is a high-level control layer for some entity
    -   It isn’t supposed to do any real work on its own
    -   It should delegate the work to the implementation layer (also called platform)
-   For example, consider a web application
    -   It has several different GUIs for different types of users (customers, admins)
        -   And supports several different APIs (to launch the app for different OS)
    -   We can create subclasses for all the combinations of GUI & API
        -   But it will grow the class heirarchy exponentially if new GUIs or APIs are added
    -   Instead, we can divide the classes into two hierarchies
        -   Abstraction: the GUI layer of the app
        -   Implementation: the operating system APIs
        -   The abstraction object controls the appearance of the app
            -   Delegating the actual work to the linked implementation object
            -   This allows to change the GUI classes without touching the API classes

## Example

```rb
# Abstraction
class Shape
  attr_reader :color

  def initialize(color)
    # Reference to the object of the implementation heirarchy
    # To which all the color related work will be delegated
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

# Client Code
def draw_red_square
  color = Red.new
  shape = Square.new(color)
  shape.draw
end
```
