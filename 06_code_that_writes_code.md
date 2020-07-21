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








