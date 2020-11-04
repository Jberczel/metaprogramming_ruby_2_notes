# Ch. 6 - Friday: Code that writes code

### Create a Class Macro (attr_checked)

Using combination of techniques learned in this book, let's create a class macro (similar to `attr_accessor`) that validates attributes.

Rather than giving all classes this class macro, we can include a module.

If the attribute doesn't pass validation, we'll throw an exception.


```ruby

class Person
  include CheckedAttributes

  attr_checked :age do |v|
    v >= 18
  end
end


me = Person.new
me.age = 39 # => OK
me.age = 17 # => Exception

```

Let's take a look at things we'll need to setup this new class macro.

### Kernel#eval

`Kernel#eval` takes a string that contains Ruby code and executes it.

```ruby
array = [1, 2]
element = 3

eval("array << element") # => [1, 2, 3]

```

When would this be helpful?  Let's look at an example below.

#### The REST Client Example

REST Client is a simple HTTP client library that comes with an intepreter:

```bash
$ restclient http://www.twitter.com

> html_first_chars = get("/")[0..14]
=> "<!DOCTYPE html>"

```

If we look at the gem's source code, we have top-level methods for HTTP Methods, which delegate to a Resource class:

```ruby

module RestClient
  class Resource
    def get(additional_headers={}, &block) # ...
    def post(payload, additional_heaers={}, &block) # ...

  ...
```

You'd expect the top-level GET method to look somethign like this:

```ruby
def get(path, *args, &b)
  r[path].get(*args, &b)
end

```

However, the GET method along with other HTTP methods are defined in one go:

```ruby
POSSIBLE_VERBS = ['get', 'put', 'post', 'delete']

POSSIBLE_VERBS.each do |verb|
  eval <<-end_eval
    def #{verb}(path, *args, &b)
      r[path].#{verb}(*args, &b)
    end
  end_eval
end

```

### Binding Objects

A _Binding_ is a whole scope packaged as an object that you can carry around.  Later, you can execute code in that scope using _eval_.

```ruby
class MyClass
  def my_method
    @x = 1
    binding
  end
end

b = MyClass.new.my_method

eval "@x", b # => 1
```

Ruby provides a predefined constant called TOPLEVEL_BINDING, which is just a binidng for the top-level scope:

```ruby

class AnotherMethod
  def my_method
    eval "self", TOPLEVEL_BINDING
  end
end

AnotherClass.new_my_method # => main

```

One gem that makes good use of _bindings_ is Pry.  Pry defines an `Object#pry` method that opens an interactive session inside the object's scope.

### The irb Example

irb is basically a program that parses standard input or file and passes each line to `eval`.

Deep within irb's source code:

```ruby
eval(statements, @binding, file, line)
```

where
- `statements` is just a line of Ruby code.
- `@binding` is whole scope packaged into an object to evaluate code in different contexts
- `file` and `line` are used to tweak stack trace in case of excpetions (see below)

```ruby
# this code raises an exception
x = 1 / 0
```

If you run `irb exception.rb`, you'll get an exception on line 2:

```bash
ZeroDivisionError: divided by 0
  from exception.rb2:in `/'
```

When irb calls `eval`, it calls it with the current filename and line number.  If you were to remove last two arguments:

```ruby
eval(statements, @binding) # , file, line)
```

You'd get something more like this:

```bash
ZeroDivisionError: divided by 0
  from /Users/jimbob/.rvm/rubies/ruby-2.0.0/lib/ruby/2.0.0/irb/workspace.rb:54:in `/'
```

### Strings of Code vs Blocks

`eval` evalutes a string of code instead of a block.  `instance_eval` and `class_eval` usually evaluate a block, but they can also evaluate a string of code:

```ruby
array = ['a', 'b', 'c']
x = 'd'

array.instance_eval "self[1] = x"

array # => ['a', 'd', 'c']
```

You can evaluate string of code or block, but as a rule of thumb, you should use block.

### The Trouble with eval()

- Strings of code don't always play well with your editor's syntax coloring.
- More difficult to read
- Security issues (code injection)

### The ERB Example





