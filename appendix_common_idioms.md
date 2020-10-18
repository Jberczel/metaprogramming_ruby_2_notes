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


## Symbol#to_proc()

##

