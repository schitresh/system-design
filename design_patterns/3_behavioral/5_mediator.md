## Mediator
- Reduce chaotic dependencies between objects
- Retrict direct communication between objects and force them to collaborate via mediator object

## Problem
- Let's say we have a dialog to create and edit customer profiles
- It contains various form controls like textfields, checkboxes, buttons, etc.
- Some form elements may interact with others
  - For example, selecting a checkbox may reveal a hidden textfield to enter more details
  - Clicking save validates the values of the fields
- If we implement this logic directly in the form elements, it will be hard to reuse them

## Solution
- The dialog class can act as the mediator since it's aware of all the sub-elements
- The form controls can notify the dialog whenever any action (like click) occur
- We can also extract common interface for all types of dialogs that can be reused

## Example
```rb
# Mediator
class Mediator
  def notify(from, to, msg)
    raise('Not Implemented')
  end
end

# Concrete Mediator
class Intercom
  def initialize
    @colleagues = Hash.new
  end

  def add_colleague(colleague)
    @colleagues[colleague.name] = colleague
    colleage.set_mediator(self)
  end

  def notify(from, to, msg)
    colleague = @colleagues[to]
    colleague.read_msg(from, msg)
  end
end

# Components
# Various classes that contain some business logic
# Has a reference to a mediator, but isn't aware of the actual class of the mediator
# Soo that it can be used with different mediators
class Colleague
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def set_mediator(mediator)
    @mediator = mediator
  end

  def send_msg(to, msg)
    @mediator.notify(self, to, msg)
  end

  def read_msg(from, msg)
    puts "From #{from}: #{msg}"
  end
end

# Client
def communicate
  maverick = Colleague.new('Maverick')
  goose = Colleague.new('Goose')

  intercom = Intercom.new
  intercom.add_colleague(goose)
  intercom.add_colleague(maverick)

  maverick.send_msg('Goose', 'Top Gun')
end
```
