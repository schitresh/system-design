## Facade
- Provides a simplified interface to a library, framework, or complex set of classes that contains a lot of moving parts
- Provides limited functionality and includes only those features that the client cares about

## Problem
- Let's say our code works with a broad set of objects that belong to a sophisticated library or framework
- We'd need to initialize all these objects, keep track of dependencies, execute methods iin the correct order
- This can result in tight coupling of the business logic of our classes and the implementation details of the library
- That will also make it hard to comprehend and maitain the code

## Solution
- Create a class that implements the workflow for the library and doesn't affect the business logic of the client

## Example
```rb
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

# Client
def compress_video(file)
  VideoCompressor.new(file).compress
end
```
