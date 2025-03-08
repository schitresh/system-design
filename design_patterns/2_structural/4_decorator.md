## Decorator
- Attaches new behaviors to objects by place them inside special wrapper objects that contain the behaviors
- Also known as Wrapper

## Problem
- Let's say we're working on a notification library which lets programs notify their users about important events
- We create a Notifier class with a few fields, a constructor, and a single send method
- It accepts a list of emails to send the notifications to
- A third party app can create and configure a notifier object once and use it to send notifications
- Later, we realize that some users want to receive SMS about critical issues
  - Other users might want to be notified on facebook or slack
  - Some user might prefer a combination of these channels, so creating subclasses may not scale well

## Solution
- Extending a class is the first thing that comes to mind to alter an object's behavior
- But inheritance has several caveats
  - Inheritance is static, the behavior of an existing object cannot be altered at runtime
  - Subclasses can have just one parent class, multiple inheritance is not supported in most languages
- One of the ways to overcome these caveats is by using Aggregation or Composition
  - One object has a reference to another and delegates it some work
  - Aggregation: Object A contains object B, B can live without A
  - Composition: Object A consists of object B, A manages life cycle of B, B can't live without A
  - Inheritance: Object B inherits behavior from object A, A itself does the work
- In this new approach, the linked helper object can be substituted with another
  - Changing the behavior of the container at runtime
  - An object can have references to multiple objects and delegate them different kinds of work
- The wrapper using this approach implements the same interface as the wrapped object
  - From the client's perspective, these objects are identical
  - This lets us cover an object in multiple wrappers adding the combined behavior

## Example
```rb
# Component
class Data
  def write
    raise('Not Implemented')
  end
end

# Concrete Component
class FileData < Data
  def write
  end
end

# Decorator
class DataDecorator < Data
  attr_accessor :data

  def initialize(data)
    @data = data
  end

  def write
    data.write
  end
end

# Concrete Decorator
class EncryptionDecorator < DataDecorator
  def write
    encrypt_data
    data.write
  end
end

class CompressionDecorator < DataDecorator
  def write
    compress_data
    data.write
  end
end

# Client
def write_data(data)
  file_data = FileData.new(data)
  encyption_data = EncyptionDecorator.new(file_data)
  compression_data = CompressionDecorator.new(encryption_data)
  compression_data.write
end
```
