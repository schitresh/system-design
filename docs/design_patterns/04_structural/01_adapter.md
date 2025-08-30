# Adapter

-   Also known as Wrapper
-   Allows objects with incompatible interfaces to collaborate

## Problem

-   Let's say we're creating a stock market monitoring app
    -   It downloads the stock data from multple sources in XML format
    -   And displays charts & diagrams for the user
-   Later, we decide to improve the app by integrating a third party analytics library
    -   But this library only works with data in JSON format

## Solution

-   Create an adapter that converts the interface of one object so that
    -   Another object can understand it
-   An adapter wraps one of the objects to hide the complexity of conversion
    -   The wrapped object isn’t even aware of the adapter
-   Adapters can also help objects with different interfaces collaborate
    -   In addition to converting data into various formats
    -   It gets an interface compatible with one of the existing objects
    -   Using this interface, the existing object safely calls the adapter’s methods
    -   Upon receiving a call, the adapter passes the request to the second object
        -   But in a format and order that the second object expects
-   Sometimes it’s even possible to create a two-way adapter
    -   That can convert the calls in both directions
-   For the stock market app, we can create XML-to-JSON adapters
    -   For every class of the analytics library that the code works with directly
    -   The code will communicate with the library only via these adapters

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

# Client Code
def state_code(location)
  address = AddressLocationAdapter.new(location)
  address.state_code
end

location = Location.new
state_code(location)
```
