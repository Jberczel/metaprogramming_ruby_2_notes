# Chapter 5 - Class Definitions

Per author, this is most advanced chapter in entire book.

## Class Definitions Demystified

Usually you think of only defining methods within a class defintion, but you can put any code you want inside:

```ruby
class MyClass
  puts "hello"
end

# => Hello
```


Class definitions also return the value of the last statement, just like methods and blocks:

```ruby
result = class MyClass
  self
end


result # => MyClass
```

Keep in mind that Classes and Modules are objects, so they can be _self_ in this case. Concept will be important later in this chapter.

### The Current Class

Wherever you are in a Ruby program, you alway have current object: self.  You will also have a current class (or module).  You will also have a current class (or module).

In some situations, you need to know the current class.  For example:

```ruby
def add_method_to(some_class)
  # TODO: define method m() on some_class
end
```

How do we get current class?  You can use _Module#class_eval_.

### class_eval()

_class_eval_ evaluates a block in the context of an existing class:

```ruby
def add_method_to(some_class)
  some_class.class_eval do 
    def m
      "hello"
    end
  end
end

add_method_to(String)  # => :m
"abc".m                # => "hello!"
```

_Module#class_eval_ is dfferent than _BasicObject#instance_eval_.  instance_eval only changes self, while class_eval  changes self AND current class.

By changing the current class, class_eval effectively reopens the class, just like the class keyword.

### Class Instance Variables

In a class definition, the role of _self_ belongs to the class itself, so the class instance variable `@my_var` belongs to the class rather than instances of this class:

```ruby
class MyClass
  @my_var = 1

  def self.read; @my_var; end
  def write; @my_var = 2; end
  def read; @my_var end 
end

obj = MyClass.new  # => #<MyClass:0x00007ffa1d937460>
obj.read           # => nil
obj.write          # => 2
obj.read           # => 2
MyClass.read       # => 1

```

In this example, we have two instance variables called `@my_var`, but they belong to different objects.  One belongs to instance of MyClass, and the other belogns to the class itself.  Remember classes are just objects.

#### Example of using class_eval() and class instance variables for testing

```ruby
class Loan
  def initialize(book)
    @book = book
    @time = Time.now
  end

  def to_s
    "#{@book.upcase} loaned on #{@time}"
  end
end
```

How do you write unit test for `#to_s` method?  The time will always change dependeding on when the book was loaned...

```ruby
class Loan
  def initialize(book)
    @book = book
    @time = Loan.time_class.now 
  end

  def self.time_class
    @time_class || Time
  end

  def to_s
    # ...
end
```

_Loan.time_class_ returns a class, and _Loan#initialize_ uses that class to get the current time. The class is stored in a Class Instance Variable named `@time_class`.

In production, `Loan` always uses the `Time` class, but for testing we can do:

```ruby
class FakeTime
  def self.now
    'Tue Jun 16 12:15:50'
  end
end

require 'test/unit'

class TestLoan < Test::Unit::TestCase
  def test_converstion_to_string
    Loan.instance_eval { @time_class = FakeTime  }
    loan = Loan.new('War and Peace')
    assert_equal 'WAR AND PEACE loaned on Tue Jun 16 12:15:50', loan.to_s
  end
end
```


### Singleton Methods

Ruby allows you to add a method to a single object.

```ruby
str = 'just a test string'

def str.title?
  self.upcase == self
end

str.title?                 # => false
str.methods.grep(/title?/) # => [:title?]
str.singleton_methods      # => [:title?]
```

You can also define a singleton method using _Object#define_singleton_method_ method.

#### The Truth About Class Methods

Classes are just objects, and class names are just constants.

Class methods are really just singleton methods of a class.

### Class Macros

#### The attr_accessor() example

```ruby
class MyClass
  def my_attribute=(value)
    @my_attribute = value
  end

  def my_attribute
    @my_attribute
  end
end

obj = MyClass.new
obj.my_attribute = 'x'
obj.my_attribute       # => 'x'
```

Writing accessors like this can get tedious.  As an alternative, you can use `Module#attr_*` methods:

```ruby
class MyClass
  attr_accessor :my_attribute
end
```

Class Macros look like keywords, but they're just regular class methods that are meant to be used in a class definition. Once again, class methods (and macros) are singleton methods.


#### Example of using class macros to deprecate old method names

```ruby
def self.deprecate(old_method, new_method)
  define_method(old_method) do |*args, &block)
    warn "Warning: #{old_method}() is deprecated. Use #{new_method}()."
    send(new_method, *args, &block)
  end
end


deprecate :GetTitle, :title
deprecate :LEND_TO_USER, :lend_to
deprecate :title2, :subtitle

```

### Singleton Classes

Objects can have their own hidden, special class, sometimes referred to as _metaclass_ or _eigenclass_.

Ruby has a speicla syntax based on the _class_ keyword that places you in the scope of the singleton class:

```ruby
class << an_object
  # your code here
end
```

If you want a reference to the singleton class, you can return self:

```ruby
singleton_class = class << obj
  self
end

# Or with Object#singleton_class
"abc".singleton_class
```

Singleton classes only have a single instance, and they can't be inherited. More importantly, it's where an object's Singtone Methods live.

#### Class Methods Syntaxes

Because class methods are just Singleton Methods that live in the class's singleton class, you have three ways to define a class method:

```ruby
def MyClass.a_class_method; end

class MyClass
  def self.another_class_method; end
end

class MyClass
  class << self
    def yet_another_class_method; end
  end
end
```

#### Singleton classes and modules

If you're trying to define a class method by including a module, this doesn't work:

```ruby
module MyModule
  def self.my_method
    'hello'
  end
end

clas MyClass
  include MyModule
end

MyClass.my_method # NoMethodError!
```

However, you can use singleton class:

```ruby
module MyModule
  # define as a regular instance method instead
  def my_method
    'hello'
  end
end


class MyClass
  class << self
    include MyModule
  end
end

MyClass.my_method # => 'hello'

```


#### Object#extend

Class extensions and Object extensions are common enough that Ruby provides a method just for them.

```ruby
module MyModule
  def my_method
    'hello'
  end
end

obj = Object.new
obj.extend MyModule
obj.my_method # => 'hello'


class MyClass
  extend MyModule
end

MyClass.my_method # => 'hello'

```

_Object#extend_ is simply a shortcut that includes a module in the receiver's singleton class.


## Method Wrappers

### Aliases

You can give an alternate name to a method using _Module#alias_method_.

```ruby
class MyClass
  def my_method
    'my_method()'
  end

  alias_method :m, :my_method
end
```

Aliases are common everywhere in Ruby.  For example, _String#size_ is an alias for _String#length_.

What happens if you alias a method and then redefine it?

```ruby
class String
  alias_method :real_length, :length
  
  def length
    real_length > 5 ? 'long' : 'short'
  end
end

"War and Peace".length      # => "long"
"War and Peace".real_length # => 13 
```

When you redefine a method, you don't really change the method.  Instead, you define a new method and attach an existing name to that new method.  You can still call the old version of the method as long as you have another name that's still attached to it.

### Refinements rather than Around Alias

```ruby

module StringRefinement
  refine String do
    def length
      super > 5 ? 'long' : 'short'
    end
  end
end

using StringRefinement
"War and Peace".length # => "long"
```

Like other Refinements, this Refinement wrapper applies only until the end of the module definition.

This makes it generally safter than using "Around Alias" technique, which applies everywhere.


### Module#prepend

_Module#prepend_ inserts the module below the includer in the chain of acenstors. This means a prepended module can override a method in the includer and call the non-overridden version with _super_:

```ruby
module ExplicitString
  def length
    super > 5 ? 'long' : 'short'
  end
end


String.class_eval do
  prepend ExplicitString
end


"War and Peace".length # => "long"
```

It's not local like a Refinement wrapper, but it's generally considered cleaner and more explicit than both a Refinement Wrapper and Around Alias.

### Ruby Syntactic Sugar Example

The + operator is syntactic sugar for a method named _Fixnum#+_.  When you write 1 + 1, it is  internally converted to 1.+(1).

As an exercise, you could redefine the + operator to return the correct result plus one.

```ruby
class Fixnum
  alias_method :old_plus, :+

  def +(value)
    self.old_plus(value).old_plus(1)
  end
end
```


