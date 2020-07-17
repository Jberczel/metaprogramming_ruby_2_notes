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

Similarly, we can use _Kernel#define_method_ and pass it a closure/block to print `my_var` inside my_method call.

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

define_methods

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

You can access local variables via flat scope:

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

## Callable Objects

Using a block is a two step process: first, you set some code aside; second, you call the block (with `yield`).

This "package code first, execute later" is not exclusive to blocks:

- procs -> basically a block turned object
- lambdas -> slight variation of a proc
- methods

### Proc Objects

```ruby
inc = Proc.new { |x| x + 1 }
# Some more code...
inc.call(2) # => 3
```

This technique is called _Deferred Evaluation_. You can do similar technique with _lambda_:

```ruby
inc = lambda { |x| x + 1 }
inc.class # => Proc
inc.call(2) # => 3
```

You can also use "stabby lambda" notation:

```ruby
inc = ->(x) { x + 1 }
inc.call(2) # => 3
```

### The & Operator

A block is like an additional, anonymous argument to a method.  In most cases, you execute block using _yield_.  But sometimes that isn't enough:

- You want to pass block to another method or even another block
- You want to convert block to a proc

The `&` operator basically converts callable object to a block.

```ruby
def math(a, b)
  yield(a, b)
end


def do_math(a, b, &operation)
  math(a, b, &operation)
end


do_math(2, 3) { |x, y| x + y } # => 5
```

And to convert back to callable object, remove the `&` operator.

```ruby
def my_method(&the_proc)
  the_proc
end

p = my_method { |name| "Hello, #{name}" }
p.class        # => Proc
p.call("John") # => "Hello, John"

```


### Procs vs Lambdas

Two main differences: how `return` keyword is interpreted and how arguments are handled.

For lambdas, `return` returns from lambda similar to methods.  For proc, it returns from the scope where the proc itself was defined.

For lambdas, need exact amount of arguments, otherwise it will throw _ArugmentError_.  Procs will ignore additional arguments. 

Generally, you should prefer using lambdas since they're more similar to methods.


## Method Objects

Methods are also callable objects:

```ruby
class MyClass
  def initialize(value)
    @x = value
  end

  def my_method
    @x
  end
end


object = MyClass.new(1)
m = object.method :my_method
m.call # => 1
```

By calling _Kernel#method_, you get the method itself and can call it later.

The main difference between a method and other callable objects is that a lambda is evaulated in the scope it's defined, while a _Method_ is evaulated in the scope of its object.


### Callable Objects Wrap-Up

- **Blocks** - evaluated in the scope wher they're defined.  Also, not really an object, but are "callable"
- **Procs** - Objects of class _Proc_. Like blocks, they are evaluated in the scope they're defined.
- **Lambdas** - Also objects of class _Proc_, but have slightly different behaivor for handling _return_ keyword and arguments passed when calling object.
- **Methods** - Bound to an object and evaluated in that object's scope.


