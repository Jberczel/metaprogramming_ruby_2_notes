`puts("hello world")` is the same as `puts "hello world"`. 

Another example using a setter:

```ruby
class C
  def my_attribute=(value)
    @value = value
  end

  def my_attribute
    @value
  end
end

obj = C.new

# Don't need the parenthesis
obj.my_attribute = 'some value'
obj.myattributed # => 'some value'
```

In this case, `obj.my_attribute=('some value')` is the same as `obj.my_attribute = 'some value'`.

Another example with integers: `1 + 1` is the same as `1.+(1)` in Ruby.

## Nil Guards

```ruby
a ||= []
```

This is equivalent to:

```ruby
a || (a = [])
```

In Ruby, any value is considered true with the exception of `nil` and `false`.

If the first value operand is true, then || simply returns first operand.

The above code has same effect as:

```ruby
if defined?(a) && a
  a
else
  a = []
end
```

### Nil Guards and Boolean Values

Nil guards don't work well with Boolean values or `nil`.

Here's an example:

```ruby
def calculate_initial_value
  puts 'calcuate...'
  false
end

b = nil
2.times do
  b ||= calculate_initial_value
end

# => calculate...
# => calculate...
```

In this case, `calculate_initial_value` is called twice, instead of once as expected.

The first operand is set to false, so the second operand is always called.


## Self Yield

When you pass a block to method, you expect method to call back to block with _yield_.

A twist on callbacks, is that you can pass object itself to the block.

### The Faraday Example


Here we initialize a new connection with a URL and block. You could also pass a hash of parameters to `farday.new`, but the block-based style makes it clear that you're focusing on the same object:

```ruby
require 'faraday'

conn = Faraday.new("https://twitter.com/search", do |faraday|
  faraday.response  :logger
  faraday.adapter Faraday.default_adapter
  faraday.params["q"] = "ruby"
  faraday.params["src"] = "typd"
end

response = con.get
response = status
```

Let's look at how it's implemeneted:

```ruby
module Faraday
  class <<
    def new(url = nil, options = {})
      # ...
      Faraday::Connection.new(url,options, &block)
    end

    # ...
```

The interesting stuff happens in `Faraday::Connection#initialize`, which accepts an optional block and _yields_ the newly created object to the block:

```ruby
module Faraday
  class Connection
    def initialize(url = nil, options = {})
      # ...
      yield self if block_given?
      # ...
    end
```

This simple idiom is known as _Self Yield_.  They're pretty common in Ruby.

### The tap() Example

It's common to see chained methods in Ruby.  In other languages, it's sometimes frowned upon (and sometimes referred to as "train wrecks").

```ruby
['a', 'b', 'c'].push('d').shift.upcase.next # => "B"
```
Due to Ruby's terse syntax, chains are generally more readable.  However, how do you track error somwhere along the chain?

Normally, you could break the chain and add print intermediate results. Or, you can use `#tap`:

```ruby
['a', 'b', 'c'].push('d').shift.tap { |x| puts x }.upcase.next # => "B"
```

If you were to write your own `#tap` method:

```ruby
class Object
  def tap
    yield self
    yield
  end
end
```

## Symbol#to_proc()

```ruby
names = ['bob', 'bill', 'heather']
names.map { |name| name.capitalize } # => ["Bob", "Bill", "Heather"]
```

This is considered a "one-call block".  It takes a single argument and calls a single method.

The idea behind _Symbol#to_proc_ is to repalce one-call blocks with a shorter construct.

First, let's represent the method as a symbol:

 `:capitalize`

We want to conver that to a block:

`{ |x| x.capitalize }`

As a first step, we could do:

```ruby
class Symbol
  def to_proc
    Proc.new { |x| x.send(self) }
  end
end
```
