## Prototype
- To copy existing objects without making the code dependent on their classes
- It's easier to copy complex object rather than constructing them from scratch

## Problem
- To copy a object, we've to copy the  alues of all the fields from the original object
- But some of the fields may be private and not visible form outside of the object
- It also becomes dependent on the class of the object

## Solution
- Delegate the cloning process to the actual objects being cloned
- Create an object of the class and carry over all the field values of the old object

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
      Marshal.load(Marshal.dump(object))
    end
end

# Client
def clone_component
  prototype = Prototype.new
  prototype.component = Component.new
  prototype.reference = Reference.new
  prototype.clone
end
```
