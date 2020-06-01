# Chapter 3 - Tuesday: Methods

## Dynamic Methods

Ruby allows you to call and define methods at runtime, which removes repetitive code such as defining getters and setters for numerous attributes.

### Calling Methods Dynamically

Normally yo do this:

```ruby
class MyClass
  def my_method(arg)
    arg * 2
  end
end

obj = MyClass.new
obj.my_method(3) # => 6
```

Alternatively, you can call _MyClass#my_method_ using _Object#send_:

```ruby
obj.send(:my_method, 3) # => 6
```

This is referred to as _Dynamic Dispath_, where you wait until the last moment to decide which method to call.

Why use symbol (rather than string) for method name? Symbols are immutable, which make them good for naming things.


You can call any method with _send_, including private methods! If you want to respect receiver's privacy, you can use `#public_send`. In the wild, most people don't, however.


### Defining Methods Dynamically

You can define a method on the spot with _Module#define_method_. This is referred to as a _Dynamic Method_.

```ruby
class MyClass
  define_method :my_method do |arg|
    arg * 3
  end
end

obj = MyClass.new
obj.my_method(3) # => 6
```

An example using _Dynamic Methods_ along with regular expression to remove duplicate code.

```ruby
class Computer
  def initilize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
    data_source.methods.grep(/^get_(.*)_info$/) { Computer.define_component $1 }
  end

  def self.define_component(name)
    define_method(name) do
      # ...
    end
  end
end
```

## method_missing

Method lookup starts with object's class and if it can't find it, it searches up the ancestor chain all the way up to _Object_ and eventually into _BasicObject_.


```ruby
class Lawyer; end
nick = Lawyer.new
nick.talk_simple # => NoMethodError: undefined method 'talk_simple'
```

If it cannot find a method, it calls `#method_missing`. It is a private instance method of _BasicObject_, which every object inherits. We can dynamically call it with _send_.

```ruby
nick.send(:method_missing, :talk_simple) # => NoMethodError: undefined method 'talk_simple'
```

### Overriding method_missing

Overriding method_missing allows you to call methods that don't exist (Ghost Methods). It's like saying "if you don't understand a message, do this."

```ruby
class Lawyer
  def method_misisng(method, *args)
    puts "You called: #{member}(#{args.join(', ')})"
    puts "You also passed it a block" if block_given?
  end
end

nick = Lawyer.new
nick.talk_simple(a, b) do
  # a block
end
```

```
You called: talk_simple(a,b)
You also passed it a block
```

### The Hashie Example

**Todo**

### Dynamic Proxies

Dynamic proxy refers to when you collect methods calls through _method_missing_ and forward them to a wrapped object.

```ruby
class Computer
  def initialize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
  end

  def method_missing
    super if !@data_source.respond_to?("get_#{name}_info")

    info = @data_source.send("get_#{name}_info", @id)
    price = @data_source.send("get_#{name}_price", @id)
    result = "#{name.capitalize}: #{info} ($#{price})"
    return "* #{result}" if price >= 100
    result
  end
end
```

What happens if you call _Computer#mouse_?  Call gets routed to _method_missing_, which checks if wrapped data source has a _get_mouse_info_ method. If it doesn't, call falls back to BasicObject#method_missing.

If the wrapped data source knwos about it, the call gets forwarded.

**TODO** Add note about `respond_to?` not recognizing ghost methods.

## Blank Slates

A blank slate refers to a "skinny" class with a minimual number of methods that hopefully won't collide with any ghost methods we forward using a dynamic proxy.

### BasicObject

_BasicObject_ only has a handful of methods, so is suitable to use as a blank slate.

```ruby
im  = BasicObject.instance_methods
im # => [:==, :equal?, :!, :!=, :instance_eval, :instance_exec, :__send__, :__id__]
```

By default, Ruby objects in herit from _Object_ (which is itself a subclass of _BasicObject_).  Inheriting from _BasicObject_ is the quicker way to define a Blank Slate in Ruby.

```ruby
class Computer < BasicObject
  # ...
```

### Removing Methods

**TODO**

### Dynamic Methods vs Ghost Methods

Use Dynamic Methods if you can and Ghost Methods if you have to.  Ghost methods are more dangerous and can lead to very tricky bugs.  

If you must use Ghost Methods, always make sure to call `super`.


