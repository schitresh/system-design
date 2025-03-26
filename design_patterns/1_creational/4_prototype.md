## Prototype
- Helps to copy existing objects without making the code dependent on their classes
- It's easier to copy complex object rather than constructing them from scratch

## Problem
- To create an exact copy of an object
  - We've to create a new object of the same class
  - Then go through all the fields and copy their values to the new object
- But not all objects can be copied this way
  - Some of the fields may be private and not visible from outside of the object
  - Also, it makes the code dependent on the object class
    - Since it's required to create the new object
  - Sometimes, we only know the interface that the object follows
    - But not its concrete class

## Solution
- Delegate the cloning process to the actual objects that are being cloned
  - An object that supports cloning is called a prototype
- Declare a common interface for all objects that support cloning
  - Without coupling the code to the object class
  - Usually contains a single clone method
- The clone method creates an object of the class
  - And carries over all the field values of the old object into the new one
  - Enables to copy private fields because most programming languages
    - Let objects access private fields of other objects of the same class
- When objects have dozens of fields and hundreds of possible configurations
  - Cloning them might serve as an alternative to subclassing

## Example
```rb
class Prototype
  attr_accessor :component, :reference

  def clone
    @component = deep_copy(@component)
    @reference = deep_copy(@reference)
    @reference.prototype = self
    deep_copy(self)
  end

  private

    def deep_copy(object)
      # The marshaling library converts collections of ruby objects into a byte stream
      # allowing them to be stored outside the currently active script. This data may
      # subsequently be read and the original objects reconstituted.
      Marshal.load(Marshal.dump(object))
    end
end

# Client Code
def clone_component
  prototype = Prototype.new
  prototype.component = Component.new
  prototype.reference = Reference.new
  prototype.clone
end
```
