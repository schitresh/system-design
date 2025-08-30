# Proxy

-   Provides a substitute or placeholder for another object
-   Controls access to the original object
    -   Allowing to perform something before or after the request get through to the original object

## Problem

-   Let's say we have a massive object that consumes a vast amount of system resources
    -   It is required from time to time but not always
-   We can implement lazy initialization
    -   Create this object only when it's required
    -   All of the object's clients would need to execute some deferred initialization code
    -   This would cause a lot of code duplication
-   We can put this code directly into the object's class but that isn't always possible
    -   For example, the class may be part of a closed third party library

## Solution

-   Create a proxy class with the same interface as the original service object
-   Update the app such that it passes the proxy object to all the original object's clients
-   On receiving a request from a client
    -   The proxy creates a real service and delegates all the work to it
-   If anything needs to be executed before or after the primary logic of the class
    -   The proxy can handle it without changing that class
    -   Lazy initialization, logging, access control, caching, etc.
-   Since the proxy implements the same interface
    -   It can be passed to any client that expects a real service object

## Example

```rb
# Subject Interface
# Declares common operations for both Subject and Proxy
class Image
  def download
    raise('Not Implemented')
  end
end

# Subject
class RealImage < Image
  def download
  end
end

# Proxy
# The most common applications are lazy loading, caching, access control, logging, etc.
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

# Client Code
def download_image(image)
  image.download
end

real_image = RealImage.new
proxy = ProxyImage.new(real_image)
download_image(proxy)
```
