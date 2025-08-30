# Memento

-   Also known as Snapshot
-   Saves and restores the previous state of an object without revealing the details of its implementation

## Problem

-   Let's say we're creating a text editor app which can also format text & insert images
-   We also decide to let users undo any operations carried out on text
    -   The direct approach will be to record the state of all objects
        -   And save it in some storage before performing any operation
    -   When a user decides to revert an action, the app fetches the lastest snapshot from the history
        -   And uses it to restore the state of all objects
-   Problems with these state snapshots
    -   Will require going over all the fields and copying their values
        -   But most objects won't let others access their data, hiding in private fields
    -   Also, changing the editor class will require changing
        -   The classes responsible for copying the state of its objects
    -   We will require a container class to will store all these states
        -   It may require giving public access to the editor attributes
        -   To allow other objects to write and read data to and from a snapshot
        -   Which may expose all this state information

## Solution

-   These problems are caused by broken encapsulation
    -   Some objects try to do more than they are supposed to
    -   To collect the data required to perform some action
        -   They invade the private space of other objects
        -   Instead of letting these objects perform the actual action
-   The Memento pattern delegates creating the state snapshots to the actual owner of that state
    -   Instead of other objects trying to copy the editor’s state from the outside
    -   The editor class itself can make the snapshot
-   Store the copy of the object’s state in a special object called memento
    -   The contents of the memento aren’t accessible to any other object except the one that produced it
    -   Other objects must communicate with mementos using a limited interface
        -   Which may allow fetching the snapshot’s metadata, but not the state
    -   Such a restrictive policy lets us store mementos inside other objects, usually called caretakers
-   For the editor, we can create a separate history class to act as the caretaker
    -   A stack of mementos stored inside the caretaker will grow
        -   Each time the editor is about to execute an operation
    -   We can even render this stack within the app’s UI displaying the history

## Example

```rb
# Originator
# Produces snapshots of its own state
# As well as restores its state from snapshots when required
class Originator
  attr_reader :state
  private :state

  def save
    Memento.new(@state)
  end

  def restore(memento)
    @state = memento.state
  end
end

# Memento
# Acts as a snapshot of the originator's state
class Memento
  attr_reader :state
  private :state

  def initialize(state)
    @state = state
  end
end

# Caretaker
# Keeps track of the originator's history by storing a stack of mementos
class Caretaker
  attr_reader :mementos
  private :mementos

  def initialize(originator)
    @mementos = []
    @originator = originator
  end

  def backup
    @mementos << @originator.save
  end

  def undo
    return if @memontos.blank?

    memento = @mementos.pop
    @originator.restore(memento)
  end

  def history
    mementos.each { |memento| puts memento.display }
  end
end

# Client Code
def perform_task_and_undo
  caretaker = Caretaker.new
  originator = Originator.new
  caretaker.backup(originator)
  perform_task
  caretaker.undo
end
```
