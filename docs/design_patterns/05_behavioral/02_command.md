# Command

-   Also known as Action, Transaction
-   Turns a request into a stand-alone object that contains all the information about the request
-   This lets us
    -   Pass requests as method arguments
    -   Delay or queue a request's execution
    -   Support undoable operations

## Problem

-   Let's say we're working on a new text editor app
    -   We want to create a toolbar with a bunch of buttons for various operations
-   We create a Button class
    -   But while all the buttons look similar they're supposed to do different things
        -   For example, submit, cancel, apply, save, open, print
    -   Also, where will the click handlers of these buttons be stored?
-   The simplest solution will be to create subclasses
    -   But that will create a lot of subclasses
    -   And there's a risk of breaking the code in these subclasses each time the Button class is modified
-   Also, some operations like copy & paste can be invoked from multiple places
    -   Like button on toolbar or via context menu or pressing Ctrl + C
    -   This will result in duplication of the operation's code to these places
        -   Or make menus dependent on buttons

## Solution

-   Principle of separation of concerns: Breaking an app into layers
    -   GUI layer should render buttons & capture inputs
    -   When clicked, it should send a request to the business logic layer
    -   Business logic layer carries out the operation like copy or paste
-   The command pattern suggests that GUI objects shouldn't send these requests directly
    -   Extract all the request details (object being called, method name, arguments) into a command class
    -   With a single method to trigger this request
    -   Command objects serve as links between GUI and business logic objects
-   The commands should implement the same interface
    -   Usually it has just a single execution method that takes no parameters
        -   How to pass the request details to the receiver?
        -   The command should be either pre-configured with this data, or capable of getting it on its own
    -   It lets us use various commands with the same request sender
        -   Without coupling it to concrete classes of commands
    -   As a bonus, we can switch command objects linked to the sender
        -   Effectively changing the sender’s behavior at runtime
-   Now we can just put a single field into the base Button class
    -   That stores a reference to a command object and make the button execute that command on a click

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
