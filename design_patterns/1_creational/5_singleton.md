## Singleton
- Ensures that a class has only one instance and provides a global access point to this instance
- Requires special attention in a multi-threaded environment
- Violates the single responsibility principle

## Problem
- Need to ensure that a class has a single instance
  - The most common reason for this is to control access to some shared resource
    - Like a database or a file
  - This also prevents the code from being scattered, keeping it within one class
- Provide a secure global access point to that instance
  - We use global variables to store some essential objects
  - They are handy but also unsafe since any code can potentially overwrite them


## Solution
- Make the default constructor private to prevent multiple instances
- Create a static creation method that acts as a constructor
  - Under the hood, it calls the private constructor
  - Creates an object and saves it in a static field
  - All following calls to this method return the cached object
- Just like a global variable, the singleton pattern
  - Allows accessing the object from anywhere in the program
  - But it also protects that instance from being overwritten by other code

## Example
```rb
# Naive Singleton
# It’s easy to implement a sloppy Singleton, just need to hide the constructor and
# implement a static creation method. But it behaves incorrectly in a multithreaded
# environment. Multiple threads can call the creation method simultaneously and get
# several instances of Singleton class.
class Scheduler
  private_class_method :new
  @instance = new

  def self.instance
    @instance
  end

  def schedule_event
  end
end

# Thread-Safe Singleton
# To fix the multi-thread problem, synchronize threads during the first creation of the
# Singleton object
class Scheduler
  private_class_method :new
  @instance_mutex = Mutex.new

  attr_reader :mode

  def initialize(mode)
    @mode = mode
  end

  def self.instance(mode)
    return @instance if @instance

    @instance_mutex.synchronize do
      @instance ||= new(mode)
    end

    @instance
  end

  def schedule_event
  end
end

# Client Code
def scheduler_mode(mode)
  scheduler = Scheduler.instance(mode)
  scheduler.mode
end

def process_threads
  Thread.new do
    scheduler_mode('a') # returns 'a'
  end

  Thread.new do
    scheduler_mode('b') # returns 'a'
  end
end
```
