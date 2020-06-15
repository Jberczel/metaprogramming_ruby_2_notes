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

In a class definition, the role of self belongs to the class itself, so the class instance variable `@my_var` belongs to the class rather than instances of this class:

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





