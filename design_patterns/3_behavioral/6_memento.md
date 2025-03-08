## Memento
- Save and restore the previous state of an object without revealing the details of its implementation
- Also known as Snapshot

## Problem
- Let's say we're creating a text editor app which can also format text & insert images
- We also decide to let users undo any operations carried out on text
  - The direct approach will be to record the state of all objects and save it in some storage before performing any operation
  - When a user decides to revert an action, the app fetches the lastest snapshot from the history
  - And uses it to restore the state of all objects
- Problems with the direct approach
  - Most objects won't let others access their data and hide it in private fields
  - Changing any editor class will require changing the classes responsible for copying the state of its objects
  - The container class that will store these states may expose all this state information
    - Since it needs to have public access to allow restoring the states of objects

## Solution
- Delegate the creation of snapshots for the states to the actual owners of the states (originator object)
- Store the copy of the object's state in a special object called memento
- The contents of the memento aren't accessible to any other object
- Other objects must communicate with mementos using a limited interface
  - Which may allow fetching the snapshot's metadata (creation time, operation name)
- This restrictive policy lets you store mementos inside other objects called caretakers
- The caretakers work with the mementos via the limited interface but can't tamper with the state stored inside them
- The caretaker will store a stack of mementos that can be rendered within the app's UI to display previous operations

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

# Client
def perform_task_and_undo
  caretaker = Caretaker.new
  originator = Originator.new
  caretaker.backup(originator)
  perform_task
  caretaker.undo
end
```
