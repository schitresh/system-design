# Flyweight

-   Also known as Cache
-   Fits more objects into the available amount of RAM by sharing common parts of state
    -   Between multiple objects instead of keeping all the data in each object

## Problem

-   Let's say we're creating a game where
    -   Players would be moving around a map and shooting each other
    -   Vast quantities of bullets, missiles, shrapnel from explosions
        -   Should fly all over the map delivering a thrilling experience
-   Each particle such as a bullet or a missile
    -   Is represented by a separate object containing plenty of data
    -   At some point, newly created particles may no longer fit into the remaining RAM

## Solution

### Analysis

-   On inspection of the Particle class, let's say we notice that
    -   The color and sprite fields consume a lot more memory than other fields
    -   And these two fields store almost identical data across all particles
    -   For example, all bullets have the same color and sprite
-   Other parts of a particle’s state are unique to each particle
    -   Such as coordinates, movement vector, speed
    -   The values of these fields change over time

### Intrinsic and Extrinsic State

-   The constant data of an object is usually called the intrinsic state
    -   It lives within the object
    -   Other objects can only read it, not change it
-   The rest of the object’s state is called the extrinsic state
    -   Which is often altered from the outside by other objects
-   Stop storing the extrinsic state inside the object
    -   Instead, pass this state to specific methods which rely on it
    -   Only the intrinsic state stays within the object that can be reused
-   As a result, we would need fewer of these objects
    -   Since they only differ in the intrinsic state
    -   Which has much fewer variations than the extrinsic

### Particle State

-   Extract the extrinsic state from the particle class
    -   Only three different objects would suffice to represent all particles
        -   A bullet, a missile, and a piece of shrapnel
    -   An object that only stores the intrinsic state is called a flyweight
    -   They should be immutable since they can be used in different contexts
-   Move the extrinsic state to the container object
    -   In this case, it's the main Game object
        -   That stores all particles in the particles field
-   To move it, we need to create several array fields
    -   For storing coordinates, vectors, and speed of each individual particle
    -   We need another array for storing references to a specific flyweight
        -   That represents a particle
    -   These arrays must be in sync
        -   So that the particle's data can be accessed using the same index
-   A more elegant solution is to create a separate context class
    -   That would store the extrinsic state along with reference to the flyweight object
    -   This would require having just a single array in the container class
-   The most memory-consuming fields have been moved to just a few flyweight objects
    -   A thousand small contextual objects can reuse a single heavy flyweight object
        -   Instead of storing a thousand copies of its data

## Example

```rb
# Flyweight
# Stores the intrinsic state shared between the individual objects
# Need only one object per tree type
# Accepts the extrinsic state via method parameters
class TreeType
  # Instrinsic state
  def initialize(name:, color:, texture:)
  end

  # Extrinsic state
  def draw(canvas, x, y)
  end
end

# Flyweight Factory
# Creates and manages flyweight objects
# Decides whether to re-use existing flyweight or create a new object
class TreeFactory
  attr_reader :tree_types

  def initialize(tree_type_states)
    # Flyweights, objects with only instrinsic state
    @tree_types = {}

    tree_type_states.map |state|
      name = state_name(state)
      @tree_types[name] = TreeType.new(state)
    end
  end

  def state_name(state)
    state.sort.join('_')
  end

  # Returns an existing flyweight with a given state or creates a new one
  def tree_type(shared_state)
    name = state_name(shared_state)
    tree_types[name] ||= TreeType.new(shared_state)
    tree_types[name]
  end
end

# Contextual Object
# Stores the extrinsic state like coordinates
# And reference field to flyweight (tree_type)
class Tree
  attr_reader :x, :y, :tree_type

  def initialize(x, y, tree_type)
    @x = x
    @y = y
    @tree_type = tree_type
  end

  def draw(canvas)
    tree_type.draw(canvas, x, y, tree_type)
  end
end

class Forest
  attr_reader :tree_factory, :trees

  def initialize(tree_type_states)
    @trees = []
    @tree_factory = TreeFactory.new(tree_type_states)
  end

  def plant_tree(x, y, tree_state)
    tree_type = tree_factory.tree_type(tree_state)
    tree = Tree.new(x, y, tree_type)
    trees.append(tree)
  end

  def draw(canvas)
    trees.each do |tree|
      tree.draw(canvas)
    end
  end
end
```
