## Singleton
- Ensures that a class has only one instance and provides a global access point to this instance
- Requires special attention in a multi-treaded environment

## Problem
- We may want to control access to some shared resource like a database or a file
- Doing this using global variables is unsafe since it can be potentially overwritten

## Solution
- Make the default constructor private to prevent multiple instances
- Create a static creation method that acts as a constructor and creates an object under the hood
- All following calls to this method should return the cached object

## Example
```rb
class Scheduler
  private_class_method :new
  @instance = new

  def self.instance
    @instance
  end

  def schedule_event
  end
end

# Thread-safe Singleton

class Scheduler
  private_class_method :new
  @instance_mutex = Mutex.new

  attr_reader :value

  def initialize(value)
    @value = value
  end

  def self.instance(value)
    return @instance if @instance

    @instance_mutex.synchronize do
      @instance ||= new(value)
    end

    @instance
  end

  def schedule_process
  end
end

# Client
def process_threads
  process1 = Thread.new { Schedular.instance('a') }
  process1 = Thread.new { Schedular.instance('b') }
  process1.join
  process2.join
end
```
