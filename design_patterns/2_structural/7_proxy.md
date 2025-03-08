## Proxy
- Provides a substitute or placeholder for another object
- Controls access to the original object
- Allows to perform something before or after the request get through to the original object

## Problem
- Let's say we have a massive object that consumes a vast amount of system resources
- It is required time to time but not always
- We can implement lazy initialization
  - Create this object only when required
  - All of the object's clients would need to execute some deferred initialization code
  - This would cause a lot of code duplication
- We can put this code directly into the object's class but that isn't always possible
  - For example, the class may be part of a closed third party library

## Solution
- Create a proxy class with the same interface as the original service object
- Update the app such that it passes the proxy object to all the original object's clients
- On receiving a request from a client, the proxy creates a real service and delegates all the work to it
- If anything needs to be executed before or after the primary logic of the class, the proxy can handle it
  - Lazy initialization, logging, access control, caching, etc.
- Since the proxy implements the same interface, it can be passed to any client that expects a real service object

## Example
```rb
# Service Interface
class Image
  def download
    raise('Not Implemented')
  end
end

# Service
class RealImage < Image
  def download
  end
end

# Proxy
class ProxyImage < Image
  def initialize(real_image)
    @real_image = real_image
  end

  def download
    return unless access_allowed?

    @real_image.download
    log_access
  end
end
```
