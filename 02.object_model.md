# Chapter 2 - Monday: The Object Model

## Open Classes

Can open standard library classes such as  String and add methods:

```ruby
class String
  def to_alphanumeric(s)
    s.gsub(/[^\w\s]/, '')
  end
end
 ```

### Inside Class Definitions

No real distniction between code that defines class and any other code. This code doesn't create three classes; it just opens the class definition.

```ruby
3.times do
  class C
    puts 'Hello'
  end
end
```

```
Hello
Hello 
Hello
```

Similarly, the following code doesn't create two classes. It reopens and defines new method:

```ruby
class D
  def x; 'x' end
end

class D
  def y; 'y' end
end
```

```
obj = D.new
obj.x       # => "x" 
obj.y       # => "y"
```

### The Money Example

money gem add set of utilitiy classes for managing money. It allows you to call `Numeric#to_money`:

```ruby
require "money"

standard_price = 100.to_money("USD")
standard_price.format  # => "$100.00"
```

How does it do that? The money gem modifies Numeric class:

```ruby
class Numeric
  def to_money(currency = nil)
    Money.from_numeric(self, currency || Money.default_currency)
  end
end
```

### The problem with Open Classes

Programmers can override original methods for a given class. This can lead to ugly side effects and bugs that are hard to track down.

Openning classes and modifying them is also referred to as "monkey patching".


## Inside the Object Model

Instance variables live in objects; methods live in classes.

Said differently, instance variables are not shared among objects; however, an object is an instance of a class and shares the same methods.

```ruby
class MyClass
  def my_method
    @v = 1
  end
end
```

```
obj = MyClass.new

obj.my_method
obj.instance_variables             # => [:@v]
```

```
obj.class.instance_methods(false)  # => [:my_method]
```

The false argument filters any inherited methods

### The Truth about Classes

Classes themselves are objects. Classes, like objects, have their own class:

```
"test string".class            # => String
String.class                   # => Class
```

A ruby class inherits from its superclass:

```
Array.superclass       # => Object
Object.superclass      # => BasicObject
BasicObject.superclass # => nil
```

BasicObject is the root of the Ruby class hierarchy.

### Modules

```
Class.class       # => Class
Class.superclass  # => Module
```

Every Class is an instance of Module. More specifically, a class is a module with additional instance methods: new, allocate, and superclass that allow you to create objects or arrange hierarchies.

### Constants

Constants are arranged in tree structure where modules (and classes) are _directories_ and constants are _files_.

```ruby
X = 'a root-level constant'

Module M
  class C
    X = 'a constant'
  end
   
  C::X  # => 'a constant'
end
```

```
M::C::X  # => 'a constant'  
::X      # => 'a root-level constant'
```

## Usinig Namespaces

When defining classes with common names, there's possibility of those classes classing with classes from different libraries.

For example, the original _Rake_ library defined classes _Task_ and _FileTask_. To prevent possible name classes, they wrapper theses classes in modules:

```ruby
module Rake
  class Task
  # ...
```

Now full name of task is _Rake::Task_. A module that exists to be container of constants is called a _Namespace_.

## What Happens When You Call a Method?

1. It finds the method (method lookup)
2. Executes method.  Needs something called _self_

### Method Lookup
**Receiver** is object you call a method on.  Example: `"test".reverse()`  `"test"` is the receiver.

**Ancestor Chain** is chain of classes going from receiver's class to its superclass, and then to it's superclass's superclass.

```ruby
t = "test"
t.class.ancestors # => [String, Comparable, Object, Kernel, BasicObject]
```
What about _Comparable_ and _Kernel_?  Those are modules which are included in the ancestor chain.

When you `include` a module, Ruby inserts in ancestor chain, right above (or after depending on how you view it) the including class itself.

You can also `prepend` a module which inserts right below (or before) the including class.

### The Kernel

Ruby has methods such as _print_ that you can call from anywhere in your code. It is actually an instance method of _Kernel_.

Every line of Ruby is executed inside an object.  The class _Object_ includes the _Kernel_, so you can call methods such as _print_ anywhere.

### Awesome Print Example

```
module Kernel
  def ap(object, options = {})
    # ...
  end
end
```

```
require 'awesome_print'

local_time = { city: "Rome", now: Time.now }
ap local_time, indent: 2
```

This produces:

```
{
  city: "Rome"
  now: 2020-05-20 12:50:35 +0100
}
```
### Method Execution

```ruby
  def my_method
    temp = @x + 1
    my_other_method(temp)
  end
```

Ruby needs to know who `@x` belongs to, and who is the _receiver_ for `my_other_method`.

### The self Keyword

Every line of codes is executed inside an object, and the current object is known as `self`.

Only one object can take role of `self` at a given time, but no objects holds it for a long time.

Every time you call a private method, it must be on the implicit receiver -- self.

```ruby
class C
  def public_method
    self.private_method
  end

  private

    def private_method; end
end


C.new.public_method # => NoMethodError
```

You could make this code work by removing the `self` keyword.


### The Top Level

As soon as you start a Ruby program, you're sitting within an object named _main_ that the Ruby interpreter created for you.

```ruby
self       # => main
self.class # => Object
```

### Refinements

Refinements are an alternative to monkey patching. Also, they're not global like monkey patching.

```ruby
module StringExtensions
  refine String do
    def to_alphanumeric
      gsub(/[^\w\s]/, '')
    end
  end
end
```

A refinement will only work if you add _using_ method to a given Ruby file:

```ruby
using StringExtensions
"my *1st refinement!".to_alphanumeric # => "my 1st refinement"
```
You can also add _using_ method within module to restrict refinement only to that module.

