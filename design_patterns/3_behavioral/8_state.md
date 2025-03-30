## State
- Lets an object alter its behavior when its internal state changes
- It appears as if the object changed its class
- Closely related to the concept of a finite state machine

## Problem
- The main idea is that a program can be in a finite number of states at any given time
  - Within any unique state, the program behaves differently
  - The program can be switched from one state to another instantaneously
  - Program may or may not switch to other states depending on the current state
  - These switching rules are called transitions, which are finite & predetermined
- Applying this approach to objects, let's say we have a Document class
  - A document can be in three states: draft, moderation, published
  - Draft to moderation: Can be moved only by the author
  - Moderation to draft: Moved if review fails
  - Moderation to published: If review passes, only by an administrator user
- State machines usually have a lot of conditional statements
  - That select the appropriate behavior depending on the current state of the object
  - As we add more states and state dependent behaviors to the Document class
    - They will keep increasing
    - Changing any transition logic may require changing conditionals in every method

## Solution
- Create new classes for all possible states of an object
  - And extract all state specific behaviors into them
- The original object (called context) stores a reference to the current state object
  - And delegates all state related work to that object
- To allow smooth transition of the context to another state
  - All state classes should follow the same interface
- This may look similar to the Strategy pattern
  - But in this state pattern, states are aware of each other and initiate transitions
  - While in strategy pattern, strategies almost never know about each other

## Example
```rb
# Context
# Maintains a reference to an instance of the current state subclass
# And delegates to it all state specific work
class TrafficLight
  attr_accessor :state
  private :state

  def initialize(state)
    transition_to(state)
  end

  def transition_to(state)
    @state = state
    @state.context = self
  end

  def show_signal
    @state.show_signal
  end
end

# State
# Provides a back reference to the context object
# Which can be used to transition the context to another state
class State
  attr_accessor :context

  def show_signal
    raise('Not Implemented')
  end
end

# Concrete States
class Red < State
  def show_signal
    puts "The traffic light is Red"
  end

  def next_signal
    yellow_signal = Yellow.new
    context.transition_to(yellow_signal)
  end
end

class Yellow < State
  def show_signal
    puts "The traffic light is Yellow"
  end

  def next_signal
    green_signal = Green.new
    context.transition_to(green_signal)
  end
end

class Green < State
  def show_signal
    puts "The traffic light is Green"
  end

  def next_signal
    red_signal = Red.new
    context.transition_to(red_signal)
  end
end

# Client Code
def transition_red_signal
  red_signal = Red.new
  traffic_light = TrafficLight.new(red_signal)
  traffic_light.show_signal
  traffic_light.next_signal
end
```
