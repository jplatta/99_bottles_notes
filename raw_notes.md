## Resources
* Martin Fowlers Refactoring
  * Find the list of code smells
* See Metz's References page
  * [repo for class](https://github.com/torqueforge/procore-2018-dec-b)

##TODO
* Record the changes to the 99bottles program and the reasons for each change
* Get a list of the code smells and their respective recipes

## Tools
* Rerun

## Days to the Course
Day 1: Refactoring
  * Shameless Green
  * Detecting Code Smells
  * Refactoring Recipes
Day 2: Refactoring/Extending
  * Polymorphism
Day 3:

## Steps
1. Code up shameless green and get all the tests to pass
1. Then we ask whether the code is open or closed to adding `six-pack`
2. DRY up shameless green by introducing a few new methods
  * As you're refactoring you just always have a wall of green
  * These methods label concepts that are implied by shameless green
  * These methods also isolate the logic we want to extend
3. Identified **Primitive Obsession**. The solution is to extract a new class.
4. Create a new `BottleNumber` class
  * When moving methods to new class, remember to provide defaults to the methods. Why?
5. Next smells repeated switch statements and magic numbers. Solve by replacing conditionals with polymorphism using inheritance.

## Refactoring recipes
* Flocking
* Extract Class

## DRY
* **Shameless Green**
* Only DRY your code to the extent that it saves costs.

#### Your initial code
```ruby
class Bottles
  def verse(bottles)
    "#{bottle_count(bottles).capitalize} #{pluralize(bottles)} of beer on the wall, " +
    "#{bottle_count(bottles)} #{pluralize(bottles)} of beer.\n" +
    "#{take_one(bottles)}" +
    "#{bottle_count(bottles-1)} #{pluralize(bottles-1)} of beer on the wall.\n"
  end

  def take_one(bottles)
    case bottles
    when 0
      'Go to the store and buy some more, '
    when 1
      'Take it down and pass it around, '
    else
      'Take one down and pass it around, '
    end
  end

  def bottle_count(bottles)
    bottles == 0 ? 'no more' : bottles.to_s
  end

  def pluralize(bottles)
    bottles == 1 ? 'bottle' : 'bottles'
  end
end
```

#### Metz's Shameless Green solution:
```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    case number
    when 0
      "No more bottles of beer on the wall, no more bottles of beer.\nGo to the store and buy some more, 99 bottles of beer on the wall.\n"
    when 1
      "1 bottle of beer on the wall, 1 bottle of beer.\nTake it down and pass it around, no more bottles of beer on the wall.\n"
    when 2
      "2 bottles of beer on the wall, 2 bottles of beer.\nTake one down and pass it around, 1 bottle of beer on the wall.\n"
    else
      "#{number} bottles of beer on the wall, #{number} bottles of beer.\nTake one down and pass it around, #{number-1} bottles of beer on the wall.\n"
    end
  end
end
```

## ETC Notes
* Rule of thumb: have three concrete instances before considering an abstraction.
* Indirection leads to code that is more mostly and difficult to grok

## SOLID
* **O** Open for extension; closed for modification.
* Refactor the code to make it open for extension before attempting to extend the code, i.e. to add six-packs.

## Code Smells
**Code Smell** is anti-pattern that has a known solution.

* Smells:
  * Speculative generality
  * Repeated switch statement
  * Magic number: number without a name
  * **Duplication**
  * Large class
  * **Primitive obsession**
  * **Shotgun Surgery** (Leaky Abstraction)
  * Data clump
* Refactorings (recipes for correcting smells):
  * Abstract class (for primitive obsession) - **Extract Class** recipe
  * Abstract method
  * Replace conditional with polymorphism (uses OO inheritance)
  * Replace conditional with state/strategy (uses OO composition)
* See the 22 smells in Fowler's book.
* Horizontal refactoring (?)

## Polymorphism

* Liskov Substitution Principle: a subclass must be substitutable for its parent class. In other words the subclasses must conform to the same API as their parent classes.
* Polymorphism: many forms/objects/classes responding to the same message. Many classes responding to the same API.

## Flocking Rules

![flocking rules](/images/flocking_rules.jpg)

1. Select the things that are most alike.
* (note: maybe should say "least different")
2. Find the smallest differences between them.
3. Make the simplest changes to remove the difference.
* Parse the new code
* Parse and execute it
* Parse, execute, and use its result
* Delete unused code

### Notes on Flocking Rules
* Started with shameless green
* Start making the fewest number of changes that make this and the line below "else" the
* When abstracting to the container method, start with "bottles" because it's more commonly used than "bottle"

```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    case number
    when 0
      "No more bottles of beer on the wall, no more bottles of beer.\nGo to the store and buy some more, 99 bottles of beer on the wall.\n"
    when 1
      "1 bottle of beer on the wall, 1 bottle of beer.\nTake it down and pass it around, no more bottles of beer on the wall.\n"
    when 2

      "#{number} bottles of beer on the wall, #{number} bottles of beer.\nTake one down and pass it around, #{number-1} #{container(number-1)} of beer on the wall.\n"
    else
      "#{number} bottles of beer on the wall, #{number} bottles of beer.\nTake one down and pass it around, #{number-1} #{container(number-1)} of beer on the wall.\n"
    end
  end

  def container(number=nil)
    if number == 1
      "bottle"
    else
      "bottles"
    end
  end
end
```

4. Finally deleted the repeated lines:
5. In order to get rid of differences between code. Name the difference and create a method to handle the branching logic, e.g. `container`

```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    case number
    when 0
      "No more bottles of beer on the wall, no more bottles of beer.\nGo to the store and buy some more, 99 bottles of beer on the wall.\n"
    when 1
      "1 bottle of beer on the wall, 1 bottle of beer.\nTake it down and pass it around, no more bottles of beer on the wall.\n"
    else
      "#{number} bottles of beer on the wall, #{number} bottles of beer.\nTake one down and pass it around, #{number-1} #{container(number-1)} of beer on the wall.\n"
    end
  end

  # Start with "bottles" because it's more commonly used than "bottle"
  def container(number=nil)
    if number == 1
      "bottle"
    else
      "bottles"
    end
  end
end
```

6. All verse types reduced to a single line:
* This code is better because it puts a name to concepts implied by the song, but yet noted and it isolates these concepts inside abstractions. The code is not yet open to six packs. The increased isolation suggests that the next step is to reduce another code smell.
* The isolation introduced with the new methods names concepts implied by the shameless green implementation.

```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    "#{label(number).capitalize} #{container(number)} of beer on the wall, " +
    "#{label(number)} #{container(number)} of beer.\n" +
    "#{action(number)}, " +
    "#{label(successor(number))} #{container(number-1)} of beer on the wall.\n"
  end

  # Start with "bottles" because it's more commonly used than "bottle"
  def container(number)
    if number == 1
      "bottle"
    else
      "bottles"
    end
  end

  def pronoun(number)
    if number == 1
      "it"
    else
      "one"
    end
  end

  def label(number)
    if number == 0
      "no more"
    else
      number.to_s
    end
  end

  def action(number)
    if number == 0
      "Go to the store and buy some more"
    else
      "Take #{pronoun(number)} down and pass it around"
    end
  end

  def successor(number)
    if number == 0
      99
    else
      number-1
    end
  end
end
```

7.  **TIP** Don't supply behavior to an object, e.g. number should know how to supply its own `label`
* **NEXT STEP** Identified **Primitive Obsession**. The solution is to extract a new class.
* **Decorator Pattern** Dynamically adding functionality to an object.
* Determine what `number` means, i.e. either the number of verses or bottle quantity
* Change state = mutate
* Make objects that never mutate.
* Automatic cache invalidation.
* **TIP** You should be bothered by blank lines in methods. It indicates a method that doesn't have a single responsibility

In order to clean up the Primitive Obsession, extract a new class `BottlishNumber` with the methods from the `Bottles` class:
```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    "#{label(number).capitalize} #{container(number)} of beer on the wall, " +
    "#{label(number)} #{container(number)} of beer.\n" +
    "#{action(number)}, " +
    "#{label(successor(number))} #{container(successor(number))} of beer on the wall.\n"
  end

  def label(number)
    BottlishNumber.new(number).label
  end

  def container(number)
    BottlishNumber.new(number).container
  end

  def action(number)
    BottlishNumber.new(number).action
  end

  def successor(number)
    BottlishNumber.new(number).successor
  end
end

class BottlishNumber
  attr_reader :number

  def initialize(number)
    @number = number
  end

  def container
    if number == 1
      'bottle'
    else
      'bottles'
    end
  end

  def action
    if number == 0
      "Go to the store and buy some more"
    else
      "Take #{pronoun(number)} down and pass it around"
    end
  end

  def label
    if number == 0
      "no more"
    else
      number.to_s
    end
  end

  def successor
    if number == 0
      99
    else
      number-1
    end
  end

  private
  def pronoun(delet_me=nil)
    if number == 1
      "it"
    else
      "one"
    end
  end
end
```

Then step-by-step replace the functionality in the `Bottles` class with the `BottlishNumber` class. Make sure you have the green wall behind you the whole way.

After going through the Extract Class steps:

```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    bottle_number = BottleNumber.new(number)
    next_bottle_number = BottleNumber.new(bottle_number.successor)

    "#{bottle_number.quantity.capitalize} #{bottle_number.container} of beer on the wall, " +
    "#{bottle_number.quantity} #{bottle_number.container} of beer.\n" +
    "#{bottle_number.action}, " +
    "#{next_bottle_number.quantity} #{next_bottle_number.container} of beer on the wall.\n"
  end
end

class BottleNumber
  attr_reader :number
  def initialize(number)
    @number = number
  end

  def quantity
    if number == 0
      "no more"
    else
      number.to_s
    end
  end

  def container
    if number == 1
      "bottle"
     else
      "bottles"
     end
  end

  def action
    if number == 0
      "Go to the store and buy some more"
    else
      "Take #{pronoun} down and pass it around"
    end
  end

  def pronoun
    if number == 1
      "it"
    else
      "one"
    end
  end

  def successor
    if number == 0
      99
    else
      number - 1
    end
  end
end
```

## Recipe: Replace conditional with polymorphism (uses OO inheritance)
![extract_class](/images/extract_class.jpg)
* i.e. replace conditional with inheritance
1. Make a class hierarchy
```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    bottle_number = bottle_number_for(number)
    next_bottle_number = bottle_number_for(bottle_number.successor)

    "#{bottle_number.quantity.capitalize} #{bottle_number.container} of beer on the wall, " +
    "#{bottle_number.quantity} #{bottle_number.container} of beer.\n" +
    "#{bottle_number.action}, " +
    "#{next_bottle_number.quantity} #{next_bottle_number.container} of beer on the wall.\n"
  end

  def bottle_number_for(number)
    case number
    when 0
      BottleNumber0
    when 1
      BottleNumber1
    else
      BottleNumber
    end.new(number)
  end
end

class BottleNumber
  attr_reader :number
  def initialize(number)
    @number = number
  end

  def quantity
    number.to_s
  end

  def container
    "bottles"
  end

  def action
    "Take #{pronoun} down and pass it around"
  end

  def pronoun
    "one"
  end

  def successor
    number - 1
  end
end

class BottleNumber0 < BottleNumber
  def quantity
    "no more"
  end

  def action
    "Go to the store and buy some more"
  end

  def successor
    99
  end
end

class BottleNumber1 < BottleNumber
  def container
    "bottle"
  end

  def pronoun
    "it"
  end
end
```

* Create a new `BottleNumber0` class
* Move methods to override down to the new class
* Create a factory method
* Repeat for `BottleNumber1` class

* Removing the factory method to somewhere it is globally available:

* You haven't thought hard enough about names if you feel compelled to use a pattern name as a name of a class
* Moving the factory method into the `BottleNumber` class
* The feeling that you want to qualify names, e.g. `bottle_quantity`, often indicates a new type

```ruby
class Bottles
  def song
    verses(99, 0)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    bottle_number = BottleNumber.for(number)
    next_bottle_number =  BottleNumber.for(bottle_number.successor)

    "#{bottle_number.quantity.capitalize} #{bottle_number.container} of beer on the wall, " +
    "#{bottle_number.quantity} #{bottle_number.container} of beer.\n" +
    "#{bottle_number.action}, " +
    "#{next_bottle_number.quantity} #{next_bottle_number.container} of beer on the wall.\n"
  end
end

class BottleNumber
  def self.for(number)
    return number if number.is_a?(BottleNumber)
    case number
    when 0
      BottleNumber0
    when 1
      BottleNumber1
    else
      BottleNumber
    end.new(number)
  end

  attr_reader :number
  def initialize(number)
    @number = number
  end

  def quantity
    number.to_s
  end

  def container
    "bottles"
  end

  def action
    "Take #{pronoun} down and pass it around"
  end

  def pronoun
    "one"
  end

  def successor
    number - 1
  end
end
```

* Successor method commits a Liskov violoation. Why?
  * It returns an object that doesn't conform to the `BottlishNumber` API even though we expect it to.
* Handling successor method:
  * Don't pass `self` to the successor method
  * Don't create a new instance of the `Bottles` class in the `BottleNumber` classes

```ruby
class BottleNumber
  def self.for(number)
    return number if number.is_a?(BottleNumber)
    case number
    when 0
      BottleNumber0
    when 1
      BottleNumber1
    else
      BottleNumber
    end.new(number)
  end

  attr_reader :number
  def initialize(number)
    @number = number
  end

  def quantity
    number.to_s
  end

  def container
    "bottles"
  end

  def action
    "Take #{pronoun} down and pass it around"
  end

  def pronoun
    "one"
  end

  def successor
    BottleNumber.for(number-1)
  end
end

class BottleNumber0 < BottleNumber
  def quantity
    "no more"
  end

  def action
    "Go to the store and buy some more"
  end

  def successor
    BottleNumber.for(99)
  end
end
```

* Monkey patching rules (adding methods to classes in the standard library)
  * Don't change the behavior of the standard library
  * Names you add, must be sufficiently unique to your domain

* Different versions of the factory method: the below rescue block is more difficult to understand, but more open than the case statement
* The register factories (versions 5 & 6) allow classes to register themselves so you don't have to, e.g., add them to the Hash or the switch statement.

```ruby
def self.for(number)
  # Version 6
  registry.find { |candidate| candidate.handles?(number) }.new(number)

  # Version 5
  @@registry.find { |candidate| candidate.handles?(number) }.new(number)

  # Version 4
  # Order matters
  # BottleNumber must come last
  [
    BottleNumber0,
    BottleNumber1,
    BottleNumber
  ].
    find { |candidate| candidate.handles?(number) }.new(number)

  # Version 3
  #Could put this in a YAML file
  Hash.new(BottleNumber).merge(
    0 => BottleNumber0,
    1 => BottleNumber1
  )[number].new(number)

  # Version 2
  begin
    const_get("BottleNumber#{number}")
  rescue
    BottleNumber
  end.new(number)

  # Version 1
  case number
  when 0
    BottleNumber0
  when 1
    BottleNumber1
  else
    BottleNumber
  end.new(number)
end

# def self.inherited(candidate)
#   @@registry ||= [BottleNumber]
#   @@registry.unshift(candidate)
# end

def self.inherited(candidate)
  register(candidate)
  @@registry ||= [BottleNumber]
  @@registry.unshift(candidate)
end

def self.registry
  @@registry ||=
end

def self.register(candidate)
end

class BottleNumber1 < BottleNumber
  register(self)
end
```

* Things that we want to vary, we have to isolate first.
  * Isolate the things we want to vary.
* Positional arguments inhibit refactoring. Keyword arguments enable refactoring. Always default to keyword arguments.
* When refactoring: the new code has to run (be green) before you delete the old code.

## What your code should look like when it's done
* SOLID
  * Single Responsibility
    * things should only have one reason to change
    * All the code in a class should be cohesive around its purpose (Rebecca's stuff about responsibility-driven-design)
  * Open/closed
  * Liskov - if objects say they're going to conform to some principle, they ought to
  * Interface Segregation
  * Dependency Inversion/Injection
* Freeman and Price (Growing OO Software Guided by Test, i.e. the GOOS book):
  * Loosely coupled
  * Highly cohesive
  * Easily Composable
  * Context Independent

## Creating the BottleVerse class
* Dependency inversion: depend on abstractions and not concretions, e.g. the injection of `BottleVerse` in `DescendingVerseSong`
  * Dependency injection is the primary way we do this.
* Write what you want your code to look like before implementing it, e.g. `verse_template.lyrics(number)`
* Class methods resist refactor. Favor instance methods over class methods.

* updating the tests
```ruby
module VerseTemplateTest
  def test_acts_like_a_verse_template
    assert_respond_to @role_player, :lyrics
  end
end

class BottleVerseTest < Minitest::Test
  include VerseTemplateTest
  def setup
    @role_player = BottleVerse
  end
end
```

## Testing
* Tests are just another part of your code that's consuming/using the program you're writing.
* Tight coupling leads to programs being untestable.
* TDD leads to writing objects that can be re-used
* Stubbing/mocking allow you to overcome tight coupling in your tests. (stubbing/mocking is more like an enabler for tight coupling. If you're stubbing/mocking you should think twice about how tightly coupled your code is.)
* Everytime you find a bug, write a failing test.
* **Mocking** You may not mock code you do not own.
* **Stubbing** You cannot stub in the object under test.

## Animal
* Use `refine` when monkey patching. Only applies where `using`

* `verse` in animal. Injected an object that is too stupid to bring its own behavior.
* Liskov violation. Sending/injecting/using objects that conform to different APIs.
  * so what you need are objects that polymorphically conform to the API.
    * Bad Idea: monkey patch `NilClass`:
```ruby
class NilClass
  def sound
    '<silence>'
  end

  def species
    '<silence>'
  end
end

class Farm
end
```
    * Better idea: Create a `SilentAnimal` class
```ruby
class SilentAnimal
  def sound
    '<silence>'
  end

  def species
    '<silence>'
  end
end

class Farm
  def verse(animal)
    animal ||= SilentAnimal.new
  end
end
```

```ruby
class SilentAnimal
  def sound
    '<silence>'
  end

  def species
    '<silence>'
  end
end

class Farm
  def initialize(animals)
    @animals = animals.map { |animal| animal || SilentAnimal.new }
  end

  def verse(animal)
    animal = animal.sound
  end
end
```

```ruby
class SafeAnimal
  def sound
    '<silence>'
  end

  def species
    '<silence>'
  end
end

class Farm
  def initialize(animals)
    @animals = animals.map { |animal| animal || SilentAnimal.new }
  end

  def verse(animal)
    animal = animal.sound
  end
end
```

* Anything you want other developers to do, ought to be embodied in code.
* The null pattern (Fowler's special case pattern, i.e. treating nil a special case). This ultimately a Liskov violation. Convert the nils to a class that conforms to the API, e.g. creating the `SilentAnimal` class.
* Call your own code, depend on your own code. Isolate your code from low level functionality, e.g. isolate Procore `LoginInformation` from `ActiveRecord`

## House
* cumulative Tale
* Case statement, duplication, and magic number
  * How to pick which code smell to work on?
    * Which code smells are there incidentally?
      * e.g. duplication would be gone if the case statement were gone
    * **Which code smell is related to the thing that you want to change?**
      * You want to isolate the thing you want to vary
* Science of Sync
  * Quad copters
  * Simple rules that after repeated application allow for the seeming emergence of complex behavior.

### Step 1 - make lines 1 and 2 the same
```ruby
class House
    def recite
      1.upto(12).collect {|i| line(i)}.join("\n")
    end

    def line(num)
      case num
      when 1..2
        "This is #{phrase(num)}the house that Jack built.\n"
      when 3
        "This is the rat that ate the malt that lay in the house that Jack built.\n"
      when 4
        "This is the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 5
        "This is the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 6
        "This is the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 7
        "This is the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 8
        "This is the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 9
        "This is the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 10
        "This is the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 11
        "This is the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 12
        "This is the horse and the hound and the horn that belonged to the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      end
    end

    def phrase(num=:deleteme)
      if num == 1
        ''
      else
        'the malt that lay in '
      end
    end
  end
```

### Step 2 - refactoring to array
```ruby
class House
    def recite
      1.upto(12).collect {|i| line(i)}.join("\n")
    end

    def line(num)
      case num
      when 1..2
        "This is #{phrase(num)}the house that Jack built.\n"
      when 3
        "This is the rat that ate #{phrase(num-1)} in the house that Jack built.\n"
      when 4
        "This is the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 5
        "This is the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 6
        "This is the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 7
        "This is the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 8
        "This is the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 9
        "This is the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 10
        "This is the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 11
        "This is the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 12
        "This is the horse and the hound and the horn that belonged to the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      end
    end

    def phrase(num=:deleteme)
      ['', 'the malt that lay in '].first(num).join('')
    end
  end
```

### Step 3 - add rat, reverse array order
```ruby
class House
    def recite
      1.upto(12).collect {|i| line(i)}.join("\n")
    end

    def line(num)
      case num
      when 4
        "This is the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 5
        "This is the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 6
        "This is the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 7
        "This is the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 8
        "This is the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 9
        "This is the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 10
        "This is the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 11
        "This is the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      when 12
        "This is the horse and the hound and the horn that belonged to the farmer sowing his corn that kept the rooster that crowed in the morn that woke the priest all shaven and shorn that married the man all tattered and torn that kissed the maiden all forlorn that milked the cow with the crumpled horn that tossed the dog that worried the cat that killed the rat that ate the malt that lay in the house that Jack built.\n"
      else
        "This is #{phrase(num)}the house that Jack built.\n"
      end
    end

    def phrase(num=:deleteme)
      ['the rat that ate ', 'the malt that lay in ', ''].first(num).join('')
    end
  end
```

### Step 4 - add everything to your array
```ruby
class House
    def recite
      1.upto(12).collect {|i| line(i)}.join("\n")
    end

    def line(num)
      "This is #{phrase(num)}the house that Jack built.\n"
    end

    def phrase(num=:deleteme)
      ['the horse and the hound and the horn that belonged', 'the farmer sowing his corn that kept', 'the rooster that crowed in the morn', 'the priest all shaven and shorn that married', 'the man all tattered and torn that kissed', 'the maiden all forlorn that milked','the cow with the crumpled horn that tossed','the dog that worried', 'the cat that killed', 'the rat that ate ', 'the malt that lay in ', ''].first(num).join('')
    end
  end
```
## Recipe: Replace conditional with polymorphism (uses OO composition)
* i.e. replace conditional with composition

### Using the House example starting with:
```ruby
class House
  DATA =
    [ "the horse and the hound and the horn that belonged to",
      "the farmer sowing his corn that kept",
      "the rooster that crowed in the morn that woke",
      "the priest all shaven and shorn that married",
      "the man all tattered and torn that kissed",
      "the maiden all forlorn that milked",
      "the cow with the crumpled horn that tossed",
      "the dog that worried",
      "the cat that killed",
      "the rat that ate",
      "the malt that lay in",
      "the house that Jack built"]
  attr_reader :data

  def initialize
    @data = DATA
  end

  def recite
    1.upto(12).collect {|i| line(i)}.join("\n")
  end

  def phrase(num=1)
    data.last(num).join(" ")
  end

  def line(num)
    "#{prefix} #{phrase(num)}.\n"
  end

  def prefix
    "This is"
  end
end
```

### Step 1 - add randomize using inheritance
* This was fast and easy
```ruby
class House
  DATA =
    [ "the horse and the hound and the horn that belonged to",
      "the farmer sowing his corn that kept",
      "the rooster that crowed in the morn that woke",
      "the priest all shaven and shorn that married",
      "the man all tattered and torn that kissed",
      "the maiden all forlorn that milked",
      "the cow with the crumpled horn that tossed",
      "the dog that worried",
      "the cat that killed",
      "the rat that ate",
      "the malt that lay in",
      "the house that Jack built"]
  attr_reader :data

  def initialize
    @data = DATA
  end

  def recite
    1.upto(12).collect {|i| line(i)}.join("\n")
  end

  def phrase(num=1)
    data.last(num).join(" ")
  end

  def line(num)
    "#{prefix} #{phrase(num)}.\n"
  end

  def prefix
    "This is"
  end

  def data
    DATA
  end
end

class RandomHouse < House
  def data
  # DATA.shuffle
  # super.shuffle
    @data ||= super.shuffle
  end
end
```

### Step 2 - output happens in the same order but starts as if a pirate said it
* You don't need to call super for the `prefix` override because you aren't changing what it returns
```ruby
class House
  DATA =
    [ "the horse and the hound and the horn that belonged to",
      "the farmer sowing his corn that kept",
      "the rooster that crowed in the morn that woke",
      "the priest all shaven and shorn that married",
      "the man all tattered and torn that kissed",
      "the maiden all forlorn that milked",
      "the cow with the crumpled horn that tossed",
      "the dog that worried",
      "the cat that killed",
      "the rat that ate",
      "the malt that lay in",
      "the house that Jack built"]
  attr_reader :data

  def initialize
    @data = DATA
  end

  def recite
    1.upto(12).collect {|i| line(i)}.join("\n")
  end

  def phrase(num=1)
    data.last(num).join(" ")
  end

  def line(num)
    "#{prefix} #{phrase(num)}.\n"
  end

  def prefix
    "This is"
  end

  def data
    DATA
  end
end

class RandomHouse < House
  def data
  # DATA.shuffle
  # super.shuffle
    @data ||= super.shuffle
  end
end

class PirateHouse < House
  def prefix
    'It be'
  end
end
```

### Step 3 - Refactor random to use Object Oriented Composition
* Can't create a random priate house without duplicating code in the `RandomHouse` class
* Subclasses that duplicates or wants to duplicate data/functionality in other subclasses, e.g. `RandomHouse` and `PirateHouse`
  * This violates single responsibility principle
  * Don't duplicate code in your subclasses
* So get rid of the class hierarchy
* Below code removes the responsibility of ordering DATA from `House`
* Call this **Object Oriented Composition**
* So this doesn't yet create a random pirate house. See **Step 4**
```ruby
class House
  DATA =
    [ "the horse and the hound and the horn that belonged to",
      "the farmer sowing his corn that kept",
      "the rooster that crowed in the morn that woke",
      "the priest all shaven and shorn that married",
      "the man all tattered and torn that kissed",
      "the maiden all forlorn that milked",
      "the cow with the crumpled horn that tossed",
      "the dog that worried",
      "the cat that killed",
      "the rat that ate",
      "the malt that lay in",
      "the house that Jack built"]
  attr_reader :data

  def initialize(orderer: OriginalOrderer.new)
    @data = orderer.order(DATA)
      # if random == true
      #   DATA.shuffle
      # else
      #   DATA
      # end
  end

  def recite
    1.upto(12).collect {|i| line(i)}.join("\n")
  end

  def phrase(num)
    data.last(num).join(" ")
  end

  def line(num)
    "#{prefix} #{phrase(num)}.\n"
  end

  def prefix
    "This is"
  end

  def data
    DATA
  end
end

# Two players of some role. What role? Well what varies? The order.
# Generally don't make echo chamber likes names
class RandomOrderer
  def order(data)
    data.shuffle
  end
end

class UnchangedOrderer
  def order(data)
    data
  end
end
```
### Step 4 - Implement the random pirate house
```ruby
class House
  DATA =
    [ "the horse and the hound and the horn that belonged to",
      "the farmer sowing his corn that kept",
      "the rooster that crowed in the morn that woke",
      "the priest all shaven and shorn that married",
      "the man all tattered and torn that kissed",
      "the maiden all forlorn that milked",
      "the cow with the crumpled horn that tossed",
      "the dog that worried",
      "the cat that killed",
      "the rat that ate",
      "the malt that lay in",
      "the house that Jack built"]
  attr_reader :data, :prefix

  def initialize(orderer: UnchangedOrderer.new, prefixer: UnchangedPrefixer.new)
    @data = orderer.order(DATA)
    @prefix = prefixer.prefix
  end

  def recite
    1.upto(12).collect {|i| line(i)}.join("\n")
  end

  def phrase(num)
    data.last(num).join(" ")
  end

  def line(num)
    "#{prefix} #{phrase(num)}.\n"
  end

  def data
    DATA
  end
end

class UnchangedPrefixer
  def prefix
    'This is'
  end
end

class PrivatePrefixer
  def prefix
    'Arggg'
  end
end

class PartialOrderer
  def order(data)
  end
end

class RandomOrderer
  def order(data)
    data.shuffle
  end
end

class UnchangedOrderer
  def order(data)
    data
  end
end
```

