# Facade

-   Provides a simplified interface to a library, framework, or complex set of classes
    -   That contains a lot of moving parts
-   Provides limited functionality
    -   Includes only those features that the client cares about

## Problem

-   Let's say our code works with a broad set of objects
    -   That belong to a sophisticated library or framework
    -   We would need to initialize all these objects
        -   Keep track of dependencies
        -   Execute methods in the correct order
-   This can result in tight coupling of
    -   The business logic of the classes and the implementation details of libraries
    -   Making it hard to comprehend and maitain the code

## Solution

-   Create a class that provides a simple interface to a complex subsystem
    -   Implements a specifc workflow of a large library
    -   Doesn't affect the business logic of the client

## Example

```rb
# Third Party Library
module MultiMediaLibrary
  class Video
    def initialize(file)
      @video = process(file)
      @audio = Audio.new(@video)
    end
  end

  class Audio
  end

  class Compressor
    def compress_video(video)
    end

    def compress_audio(audio)
    end
  end
end

# Facade
class VideoCompressor
  def compress(file)
    video = MultiMediaLibrary::Video.new(file)
    MultiMediaLibrary::Compressor.new.compress_video(video)
  end
end

# Client Code
def compress_video(file)
  VideoCompressor.new(file).compress
end
```
