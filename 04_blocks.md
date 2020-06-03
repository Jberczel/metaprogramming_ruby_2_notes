# Chapter 4 - Blocks

Blocks are powerful tool for controlling scope (which variables and methods can be seen by which lines of code).

Blocks are one member of larger family of "callable objects".


## Basics of Blocks

```ruby
def a_method(a,b)
  a + yield(a,b)
end

a_method(1,2) { |x, y| (x + y) * 3  } # => 10

```

You can only define block when you call a method.  The block is passed straight into the method, and the method can call back to the block with the `yield` keyword.

Optionally, a block can have arguments.  Also, like methods, blocks return reslut of last line of code it evaluates.


## Blocks are Closures

Code that runs is made up of two things: the code itself and set of bindings.  When code runs, it has an environment (variables or you can call them bindings).

```ruby
def my_method
  x = "Goodbye"
  yield("cruel")
end

x = "Hello"
my_method { |y| "#{x}, #{y} world" } # => "Hello, cruel world"

```

When you define a block, you capture local bindings (such as `x` above). Then you pass block to method which has its own separate set of bindings.  The code above only sees the `x` that was around when the block was created, not the method's `x`.

## Scope

Scope refers to what variables (local, constants, global) are available to you in the current environment.  You can think of scope as buckets and variables as marbles.

### Scope Gates

Three places where program leaves previous scope and enter new one:

- Class definitions
- Module definitions
- Methods

The broders are marked by these keywords: `class`, `module`, `def`. Each of these acts as a Scope Gate.


### Flattening the Scope

```ruby
my_var = "Success"

class MyClass
  # We want to print my_var here...
  
  def my_method
    # ... and here
  end
end

```

Rather than using the _class_ Scope Gate, you can use a method call.  Specifically for creating a new class, you can use `Class.new` and pass it a closure/block that binds to local variable.

```ruby
MyClass.new do 
  # Now we can print my_var
  puts "#{my_var} in the class definition!"

  def my_method
    # .. but how can we print it here?
  end
end
```

Similarly, we can use _Kernel#defin_method_ and pass it a closure/block to print `my_var` inside my_method call.

```ruby
  define_method :my_method do
    "#{my_var} in teh method"
  end
```

### Sharing the scope

```ruby
def define_methods
  shared = 0

  Kernel.send :define_method, :counter do
    shared
  end

  Kernel.send :define_method, :inc do  |x|
    shared += x
  end
end

counter # => 0
inc(4)
counter # => 4

```

In this example, we use dynamic dispatch and dynamic methods to create methods that have access to a "private" variable called: `shared`.

This example is also a closure, since these two methods bind to local variable and carry it arround. The `shared` variable is not accessible outside of this scope.

### instance_eval()

_BasicObject#instance_eval_ evaules a block in the context of an object:

```ruby
class MyClass
  def initialize
    @v = 1
  end
end

obj = MyClass.new

ob.instance_eval do
  self # => #<MyClass: 0x3340dc @v=1>
  @v   # => 1
end
```

You can acess local variables via flat scope:

```ruby
v = 2
obj.instance_eval { @v = 2 }
obj.instance_eval{ @v } # => 2
```

This is referred to as a _Context Probe_ where you dip inside an object.

_instance_eval_ has a more flexible twin brother _instance_exec_ that allows you to pass arguments to the block.

Don't these methods break encapsulation? Yes, and it comes with risks, but possibly good for testing or for when encapsulation gets in the way.

### Clean Rooms

When you create an object to evaluate blocks inside it.

The ideal Clean Room doesn't have many instance mehtods or instance variables, which makes _BasicObject_ good for setting up a Clean Room.

### Callable Objects



