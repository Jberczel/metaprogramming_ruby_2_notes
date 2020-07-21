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

### Kernel#eval





