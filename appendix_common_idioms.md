# Common Idioms

## Mimic Methods

Method calls in disguise. Depending on the Programmer's intention, you can utilize Ruby's flexible sytnax to make the code cleaner or to make a method call look like a keyword.

In standard Ruby library, mimic methods are used for access modifiers such as `prviate` and `protected` as well as class macros such as `attr_reader`.

One simple example of a mimic method:
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

## Self Yield

## Symbol#to_proc()

##

