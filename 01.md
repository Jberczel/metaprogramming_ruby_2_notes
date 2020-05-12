# Chapter 1 - The M Word

## Ghost Towns and Marketplaces

At runtime, a compiled language is more like a ghost town, where variables and methods have lost their concreteness.

However, an inteprated language is more like a busy marketplace, where you can ask questions.


```ruby
class Greeting
  def initialize(text)
    @text = text
  end

  def welcome
    @text
  end
end

```

Introspection refers to looking at a language construct during runtime.

```
my_object = Greeting.new("Hello")
my_object.class                          # => Greeting
my_object.class.instance_methods(false)  # => [:welcome]
my_object.instance_variables             # => [:@text]

```
This is only half the picture.  You can also write language constructs at runtime.  This is metaprogramming, where you write code that writes code.


