# Ruby POOD Training (December 2018)

## Sandi Metz Course
[course repo](https://github.com/torqueforge/procore-2018-dec-b/tree/bottles)

`$ rerun -x /path/to/test.rb`

### Open/Closed
* Is the code **Open** to change?
  * _Yes_ -> **Make the Change**
  * _No_ -> Do you know how to make it open?
    * _Yes_ -> **Make it Open**
    * _No_ -> Remove the easiest to fix or best understood **Code Smell**
      * Restart the process...

## Recipes

### Flocking Rules

1. Select the things that are most alike.
* (note: maybe should say "least different")
2. Find the smallest differences between them.
3. Make the simplest changes to remove the difference.
* Parse the new code
* Parse and execute it
* Parse, execute, and use its result
* Delete unused code

### Extract Class
1. Create a new class
* Choose a class name
* Copy methods that obsess on the primitive
* Add `attr_reader` and `initialize` to save teh primitive as data
2. Hook the new class into the old
* In every copied method of old class, forward the message to the new class.
3. Remove arguments from methods in the new class.
* Change the name of the parameter in new class
* Remove the argument from the senders one by one
* Remove the parameter and default
4. In `#verse` create temp variable for `bottle_number` and `next_bottle_number`
5. Remove the obsolete methods from `Bottles`

### Removing Parameters
1. Alter the method definition to change the argument name, and provide a default.
2. Change every sender of the message to remove the parameter.
3. Delete the argument from the method definition.

## Shameless Green
The shameless green solution might look like bad coding, but it meets the requirements at minimal cost.
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

## Shamelss to DRY
- **Code Smell** Repeated Code
- **Recipe** Flocking Rules

First ask whether shameless green is whether its open/closed to adding `six-pack`. It's not open. At this point the we decided to DRY up the code by applying the Flocking Rules hoping it would make the code open to including `six-pack`.
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
## DRY to BottleNumber
- **Code Smell** Primitive Obsession
- **Recipe** Extract Class

We've DRY-ed up the code, but it still isn't open to `six-pack`. So we move onto the next code smell. Metz reasons:

"Each item above acts like a vote, and these votes combine to point to Primitive Obsession as the dominant code smell. Built-in data classes like String , Integer , Array , and Hash are examples of "primitives." Primitive Obsession is when you use one of these data classes to represent a concept in your domain. Obsessing on a primitive results in code that passes built-in types around, and supplies behavior for them."

To remove the Primitive Obsession, extract a new class, `BottleNumber`, using the above recipe.

Remember, in order to keep the tests green the whole way, you'll first need each method in the `Bottles` class to create a new instance of `BottleNumber`. For example:
```ruby
class Bottles
  #...

  def label(number)
    BottlishNumber.new(number).label
  end

  #...
end

class BottlishNumber
  attr_reader :number

  def initialize(number)
    @number = number
  end

  #...

  def label
    if number == 0
      "no more"
    else
      number.to_s
    end
  end

  #...
end
```

Following the **Extract Class** recipe yields:
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

## Code Smell: Data Clump
Just include a method in `BottleNumber` for `#description` that returns `label` and `quantity`.

## Conditional to Polymorphism
- **Code Smell** Magic Numbers
- **Recipe** Replace Conditionals with Polymorphism

The code is looking better, but it's still not open to `six-pack`. The next code smells to tackle are the repeated switch statements and **Magic Numbers**. A magic number is a number without a name. Both of these smells suggest an underlying abstraction. Remove the smells by introducing a class hierarchy for `BottleNumber`.

Steps:
1. Create a new class, e.g. `BottleNumber1` that inherits from `BottleNumber`
2. Add the specializations to `BottleNumber1`, i.e. the methods `#container` and `#pronoun`
3. Create the `#bottle_number_for` factory method in `Bottles`.
4. Splice in use of `bottle_number_for` to the verse strings
5. Remove the conditionals from `BottleNumber`

Resulting code should look like:
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

## Successor Method on BottleNumber
- **Code Smell** Liskov Violation
- **Recipe** Make successor return a BottleNumer object

The current implementation of `#successor` violates the Liskov Substition Principle because it returns an object that doesn't conform the `BottleNumber` API. To solve this. Move the factory method from `Bottles` down to `BottleNumber`. Then have `#successor` return a new `BottleNumber` object.
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

  #...

  def successor
    BottleNumber.for(number-1)
  end
end

class BottleNumber0 < BottleNumber
  #...
  def successor
    BottleNumber.for(99)
  end
end
```

## Creating factories open for extension
- **Code Smell** Liskov Violation
- **Recipe** Make the factory globally accessible

Currently the `BottleNumber` factory is not open for extension. Here are a few ways to make it open:
### Using Try/Catch
```ruby
class BottleNumber
  def self.for(number)
    begin
      const_get("BottleNumber#{number}")
    rescue NameError
      BottleNumber
    end.new(number)
  end

  #...
end
```
### Using a Hash
```ruby
class BottleNumber
  def self.for(number)
    Hash.new(BottleNumber).merge( 0 => BottleNumber0,
                                  1 => BottleNumber1)[number].new(number)
  end

  #...
end
```
### Using the Registry
**V1 Registry**
```ruby
class BottleNumber
  def self.for(number)
    @@registry.find {|candidate| candidate.handles?(number)}.new(number)
    [BottleNumber1, BottleNumber0, BottleNumber].find {|candidate| candidate.handles?(number)}.new(number)
  end

  def self.handles?(number)
    true
  end

  def self.inherited(candidate)
    register(candidate)
  end

  #...
end

class BottleNumber0 < BottleNumber
  register(self)

  def self.handles(number)
    number == 0
  end

  #...
end

class BottleNumber1 < BottleNumber
  register(self)

  def self.handles(number)
    number == 1
  end

  #...
end
```
**V2 Registry**
```ruby
class BottleNumber
  def self.for(number)
    registry.find {|candidate| candidate.handles?(number)}.new(number)
  end

  def self.handles?(number)
    true
  end

  def self.registry
    @@registry ||= [BottleNumber]
  end

  def self.register(candidate)
    registry.unshift(candidate)
  end

  #...
end

class BottleNumber0 < BottleNumber
  register(self)

  def self.handles(number)
    number == 0
  end

  #...
end

class BottleNumber1 < BottleNumber
  register(self)

  def self.handles(number)
    number == 1
  end

  #...
end
```

## Add Six-Pack
The code is now open to `six-pack`. Create a new class and update the tests.
```ruby
class BottleNumber
  def self.for(number)
    case number
    when 0
      BottleNumber0
    when 1
      BottleNumer1
    when 6
      BottleNumber6
    else
      BottleNumber
    end.new(number)
  end

  #...
end

class BottleNumber6 < BottleNumber
  def quantity
    "1"
  end

  def container
    "six-pack"
  end
end
```

### Update the test
```ruby
class BottlesTest < Minitest::Test
  # ...
  def test_the_whole_song
    expected = <<-SONG
    99 bottles of beer on the wall, 99 bottles of beer.
    Take one down and pass it around, 98 bottles of beer on the wall.

    # ...

    7 bottles of beer on the wall, 7 bottles of beer.
    Take one down and pass it around, 1 six-pack of beer on the wall.

    1 six-pack of beer on the wall, 1 six-pack of beer.
    Take one down and pass it around, 5 bottles of beer on the wall.

    # ...

    No more bottles of beer on the wall, no more bottles of beer.
    Go to the store and buy some more, 99 bottles of beer on the wall.
    SONG

    assert_equal expected, bottles.song
  end
end
```

## Vary Verse
- `Bottles` violates the **Single Responsibility Principle**

Refactor `Bottles` into two new classes: `DescendingVerseSong` and `BottleVerse`. This new arrangement injects `BottleVerse` into `DescendingVerseSong`. This allows `DescendingVerseSong` to use `BottleVerse` as a template for a verse of the song.
```ruby
# ...
class DescendingVerseSong
  attr_reader :verse_template, :max, :min
  def initialize(verse_template: BottleVerse, max: 99, min: 0)
    @verse_template = verse_template
    @max = max
    @min = min
  end

  def song
    verses(max, min)
  end

  def verses(upper, lower)
    upper.downto(lower).map { |i| verse(i) }.join("\n")
  end

  def verse(number)
    verse_template.lyrics(number)
  end
end

class BottleVerse
  def self.lyrics(number)
    new(BottleNumber.for(number)).lyrics
  end

  attr_reader :bottle_number
  def initialize(bottle_number)
    @bottle_number = bottle_number
  end

  def lyrics
    "#{bottle_number} of beer on the wall, ".capitalize +
    "#{bottle_number} of beer.\n" +
    "#{bottle_number.action}, " +
    "#{bottle_number.successor} of beer on the wall.\n"
  end
end
# ...
```

### Update the tests to handle refactored code
```ruby
require_relative '../../test_helper'
require_relative '../lib/bottles'

class BottleVerseTest < Minitest::Test
  def test_the_first_verse
    expected = "99 bottles of beer on the wall, " +
      "99 bottles of beer.\n" +
      "Take one down and pass it around, " +
      "98 bottles of beer on the wall.\n"
    assert_equal expected, BottleVerse.lyrics(99)
  end

  def test_another_verse
    expected = "3 bottles of beer on the wall, " +
      "3 bottles of beer.\n" +
      "Take one down and pass it around, " +
      "2 bottles of beer on the wall.\n"
    assert_equal expected, BottleVerse.lyrics(3)
  end

  def test_verse_2
    expected = "2 bottles of beer on the wall, " +
      "2 bottles of beer.\n" +
      "Take one down and pass it around, " +
      "1 bottle of beer on the wall.\n"
    assert_equal expected, BottleVerse.lyrics(2)
  end

  def test_verse_1
    expected = "1 bottle of beer on the wall, " +
      "1 bottle of beer.\n" +
      "Take it down and pass it around, " +
      "no more bottles of beer on the wall.\n"
    assert_equal expected, BottleVerse.lyrics(1)
  end

  def test_verse_0
    expected = "No more bottles of beer on the wall, " +
      "no more bottles of beer.\n" +
      "Go to the store and buy some more, " +
      "99 bottles of beer on the wall.\n"
    assert_equal expected, BottleVerse.lyrics(0)
  end
end

class VerseDouble
  def self.lyrics(number)
    "This is verse #{number}.\n"
  end
end

class DescendingVerseSongTest < Minitest::Test
  def test_a_verse
    expected = "This is verse 98.\n"
    assert_equal expected, DescendingVerseSong.new(verse_template: VerseDouble).verse(98)
  end

  def test_a_couple_verses
    expected =
      "This is verse 99.\n" +
      "\n" +
      "This is verse 98.\n"
    assert_equal expected, DescendingVerseSong.new(verse_template: VerseDouble).verses(99, 98)
  end

  def test_the_whole_song
    expected =
      "This is verse 5.\n" +
      "\n" +
      "This is verse 4.\n" +
      "\n" +
      "This is verse 3.\n" +
      "\n" +
    "This is verse 2.\n"
    assert_equal expected, DescendingVerseSong.new(verse_template: VerseDouble, max: 5, min: 2).song
  end
end
```

## Handling Nils
- **Code Smell** Liskov Violation
- **Recipe** Nil Pattern

The objects need to conform to the same API even in the case that `Animal` is `nil`. So create a new class that conforms to the `Animal` API to handle the `nil` case.

### V1 - Monkey Patch NilClass (Bad)
```ruby
using Article

class NilClass
  def sound
    '<silence>'
  end

  def species
    '<silence>'
  end
end

class Farm
  # ...
end
```

### V2 - Create SilentAnimal class (Good)
```ruby
using Article

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

## Polymorphism with Composition
- **Code Smell** Single Responsibility Violation
- **Recipe** Composition

The example given uses the `RandomHouse` and `PirateHouse` classes. The class hierarchy version of the `House` classes violaed the **Single Responsibility Principle** because duplicated the same code in the subclasses.

This is the final version using polymorphism with composition:
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

## Other Notes
* Things that we want to vary, we have to isolate first.
* Depend on abstractions, not data/concretions, e.g. using dependency injection
* The feeling that you want to qualify names, e.g. `bottle_quantity`, often indicates a new type
* SOLID
* GOOS book
