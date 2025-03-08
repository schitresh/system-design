## Flyweight
- Fits more objects into the available amount of RAM by sharing common parts of state
  - Between multiple objects instead of keeping all the data in each object

## Problem
- Let's say we're creating a game where players would be mobing around a map and shooting each other
- Vast quantities of bullets, missiles, shrapnel from explosions should fly all over the map
- Each particle like bullet, missile is represented by a separate object containing plenty of data
- But the game chrashes after a while on computers with less RAM

## Solution
- We observe that the all bullets have almost identical data like color, sprite
- Only some data like coordinates, movement vector and speed are unique to each particle
  - The constant data that is only readable by other objects is called intrinsic state
  - The changing data that can be altered by other objects is called extrinsic state
- We can maintain one object per particle type with intrinsic state called flyweight objects
- And extract out the extrinsic states
  - This extrinsic state will require several array fields for storing coordinates, vectors & speed of each particle
  - It will also require an array for storing references to a specific flyweight that represents a particle
  - These arrays must be in sync to access all data of a particle using the same index
- A better way is to create a separate context class to store the extrinsic state along with references to the flyweight objects
  -

## Example
```rb
# Flyweight
# Stores the intrinsic state shared between the individual objects
class TreeType
  def initialize(name:, color:, texture:)
  end
end

# Flyweight Factory
# Creates and manages flyweight objects
class TreeFactory
  attr_reader :tree_types

  def initialize(tree_type_states)
    @tree_types = {}

    tree_type_states.map |state|
      name = state_name(state)
      @tree_types[name] = TreeType.new(state)
    end
  end

  def state_name(state)
    tree_type.sort.join('_')
  end

  def tree_type(shared_state)
    name = state_name(shared_state)
    tree_types[name] ||= TreeType.new(shared_state)
    tree_types[name]
  end
end

# Context
# Stores the extrinsic state like coordinates
# And reference field to flyweight (tree_type)
class Tree
  def initialize(x, y, tree_type)
  end
end

class Forest
  attr_reader :tree_type_factory

  def initialize(tree_type_states)
    @trees = []
    @tree_factory = TreeFactory.new(tree_type_states)
  end

  def plant_tree(x, y, tree_state)
    tree_type = tree_type_factory.tree_type(tree_state)
    tree = Tree.new(x, y, tree_type)
    trees.append(tree)
  end
end
```
