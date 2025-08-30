## Mediator
- Also known as Intermediary, Controller
- Reduces chaotic dependencies between objects
- Retricts direct communication between objects and force them to collaborate via mediator object

## Problem
- Let's say we have a dialog for creating and editing customer profiles
  - It contains various form controls like textfields, checkboxes, buttons, etc.
- Some form elements may interact with others. For example:
  - Selecting a checkbox may reveal a hidden textfield to enter more details
  - Clicking save validates the values of the fields
- If we implement this logic directly in the form elements, it will be hard to reuse them

## Solution
- Cease all direct communication between the components which need to be independent
  - These components must collaborate indirectly by calling a special mediator object
    - That redirects the calls to appropriate components
  - The components depend only on a single mediator class instead of being coupled to a dozen colleagues
- Here, the dialog class itself may act as the mediator
  - The dialog class is already aware of all of its sub-elements
  - So we won’t even need to introduce new dependencies into this class
- The submit button will no longer have to validate the values of form elements
  - Now its single job is to notify the dialog about the click
  - Upon receiving this notification, the dialog itself performs the validations
    - Or passes the task to the individual elements
  - Instead of being tied to the form elements, the button is only dependent on the dialog class
- We can go further and make the dependency even looser
  - By extracting the common interface for all types of dialogs
  - The interface would declare the notification method which all form elements can use
    - To notify the dialog about events happening to those elements
  - Thus, the submit button would be able to work with any dialog that implements that interface

## Example
```rb
# Mediator
# Interface to declare a method used by components to notify the mediator
# about various events. The Mediator may react to these events and pass
# the execution to other components.
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
# Various classes that contain some business logic. Has a reference to a mediator,
# but isn't aware of the actual class of the mediator so that it can be used with
# different mediators.
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

# Client Code
def communicate
  maverick = Colleague.new('Maverick')
  goose = Colleague.new('Goose')

  intercom = Intercom.new
  intercom.add_colleague(goose)
  intercom.add_colleague(maverick)

  maverick.send_msg('Goose', 'Top Gun')
end
```
