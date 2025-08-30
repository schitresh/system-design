# Decorator

-   Also known as Wrapper
-   Attaches new behaviors to objects
    -   By placing them inside special wrapper objects that contain the behaviors

## Problem

-   Let's say we're working on a notification library
    -   It lets programs notify their users about important events
-   We create a Notifier class with a few fields, a constructor, and a single send method
    -   The method accepts a message and sends it to a list of emails passed via constructor
    -   A third party app can create and configure a notifier object once
        -   And use it to send notifications
-   Later, we realize that some users want to receive SMS about critical issues
    -   Other users might want to be notified on facebook or slack
-   We can extend the Notifier class and create new subclasses
    -   This will require the client to instantiate the required notification class
    -   But some users might prefer a combination of these channels
    -   This will bloat both the library and the client code

## Solution

### Inheritance

-   Extending a class is the first thing that comes to mind to alter an object's behavior
-   However, inheritance has several caveats
    -   Inheritance is static
        -   The behavior of an existing object cannot be altered at runtime
    -   Subclasses can have just one parent class
        -   Multiple inheritance is not supported in most languages
-   One of the ways to overcome these caveats is by using Aggregation or Composition

### Aggregation or Composition

-   Both of the alternatives work almost the same way
    -   One object has a reference to another and delegates it some work
-   Inheritance
    -   Object B inherits behavior from object A
    -   A itself does the work
-   Aggregation
    -   Object A contains object B
    -   B can live without A
-   Composition
    -   Object A consists of object B
    -   A manages life cycle of B, B can't live without A
-   With this new approach, the linked helper object can be substituted with another
    -   Changing the behavior of the container at runtime
    -   An object can use the behavior of various classes
        -   Having references to multiple objects and delegate them different kinds of work

### Wrapper

-   Wrapper is the alternative name for the Decorator pattern
    -   That clearly expresses the main idea of the pattern
    -   A wrapper is an object that can be linked with some target object
    -   The wrapper contains the same set of methods as the target
        -   And delegates to it all requests it receives
    -   However, the wrapper may alter the result by doing something
        -   Either before or after it passes the request to the target
-   When does a simple wrapper becomes the real decorator
    -   The wrapper implements the same interface as the wrapped object
    -   That’s why from the client’s perspective these objects are identical
    -   Make the wrapper’s reference field accept any object that follows that interface
    -   This will let us cover an object in multiple wrappers
        -   Adding the combined behavior of all the wrappers to it

### Notifications

-   Leave the simple email notification behavior inside the base Notifier class
    -   But turn all other notification methods into decorators
-   The client code would need to wrap a basic notifier object
    -   Into a set of decorators that match the client’s preferences
    -   The resulting objects will be structured as a stack
-   The last decorator in the stack
    -   Would be the object that the client actually works with
    -   Since all decorators implement the same interface as the base notifier
        -   The rest of the client code won’t care whether it works with
            -   The pure notifier object or the decorated one
-   The same approach can be applied to other behaviors
    -   Such as formatting messages or composing the recipient list
    -   The client can decorate the object with any custom decorators
        -   As long as they follow the same interface as the others

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

# Client Code
# The client code can support both simple components as well as decorated ones
def write_data(data)
  file_data = FileData.new(data)
  encyption_data = EncyptionDecorator.new(file_data)
  compression_data = CompressionDecorator.new(encryption_data)
  compression_data.write
end
```
