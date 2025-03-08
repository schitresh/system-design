## Command
- Turns a request into a stand-alone object that contains all the information about the request
- Pass requests as method arguments, delay or queue a request's execution, support undoable operations

## Problem
- Let's say we're working on a new text editor app
- We want to create a toolbar with a bunch of buttons for various operations
- We create a Button class
  - But while all the buttons look similar they're supposed to do different things
  - For example, submit, cancel, apply, save, open, print
  - Where will the click handlers of these buttons be stored?
- The simplest solution will be to create subclasses
  - But that will create a lot of subclasses
  - And there's a risk of breaking the code in these subclasses each time the Button class is modified
- Also, some operations like copy & paste can be invoked from multiple places
  - Button on toolbar or via context menu or pressing Ctrl + C
  - This will result in duplication of the operation's code to these places

## Solution
- Principle of separation of concerns: Breaking an app into layers. Example:
  - GUI layer should render buttons & capture inputs
  - When clicked, it should send a request to the business logic layer
  - Business logic layer carries out the operation like copy or paste
- The command pattern suggests that GUI objects shouldn't send these requests directly
  - Extract all the request details (object being called, method name, arguments) into a command class
  - With a single method to trigger this request
- Command objects serve as links between GUI and business logic objects
  - The command parameters should either be pre-configured or the command object should get it
- The Button class can store a reference to a command object and make the button execute it on a click
-

## Example
```rb
# Command
class Command
  def initialize(app:, editor:)
    @app = app
    @editor = editor
  end

  def backup
    @backup = @editor.text
  end

  def undo
    @editor.text = @backup
  end

  def execute
    raise('Not Implemented')
  end
end

# Concrete Command
class Copy < Command
  def execute
    @app.clipboard = editor.get_selection
  end
end

class Cut < Command
  def execute
    backup
    @app.clipboard = editor.get_selection
    editor.delete_selection
  end
end

class Paste < Command
  def execute
    backup
    editor.replace_selection(@app.selection)
  end
end

class Undo < Command
  def execute
    @app.undo
  end
end

# Receiver
class Editor
  def get_selection
  end

  def delete_selection
  end

  def replace_selection
  end
end

# Invoker
class Application
  def initialize
    @editor = Editor.new
  end

  def copy
    Copy.new(self, @editor).execute
  end
end
```
