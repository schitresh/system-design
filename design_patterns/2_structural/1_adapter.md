## Adapter
- Allows objects with incompatible interfaces to collaborate

## Problem
- Let's say we're creating a stock market monitoring app
- It downloads the stock data from multple sources in XML format and displays charts & diagrams for the user
- Later, we decide to improve the app by integrating a third party analytics library
- But this library works with data only in JSON format

## Solution
- Create an adapter that converts the interface of the objects so that another objects can understand
- Adapter wraps these objects to hide the complexity of conversion
- Sometimes it's even possible to create a two way adapter that can convert the calls in both directions

## Example
```rb
# Target
class Address
  def state_code
  end
end

# Adaptee
# Needs some adaptation before the client code can use it
class Location
  def state
  end
end

# Adapter
# Makes the adaptee's interface compatible with the target's interface
class AddressLocationAdapter < Address
  def initialize(location)
    @location = location
  end

  def state_code
    state_code_from_state(@location.state)
  end
end

# Client
def state_code_from_location
  location = Location.new
  address = AddressLocationAdapter.new(location)
  address.state_code
end
```
