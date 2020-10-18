
In Rails, you can include modules and you gain both instance and class methods.  How does that happen?

_ActiveSupport::Concern_ allows you to add both and makes it easy to roll that functionality into other modules.

## Rails Before Concern

_ActiveRecord::Base_ is an assembly of dozens of modules that define both instance methods and class methods.

Before _ActiveSupport::Concern_ there was another way to roll those methods into _Base_.

## The Include-and-Extend Trick

In Rails 2, all validation methods were defined in _ActiveRecord::Validations_ (there was no Active Model at this time).

```ruby
module ActiveRecord
  module Validations
    # ...

    def self.included(base)
      base.extend ClassMethods
      # ...
    end

    module ClassMethods
      def validates_length_of(*attrs) # ...
      # ...
    end

    def valid?
      # ...
    end

    # ...
  end
end
```

When _ActiveRecord::Base_ includes _Validations_, three things happen:

1. Instance methods such as `#valid?` become instance methods of _Base_.
2. Ruby calls the _included_ Hook Method on _Validations_, passing _ActiveRecord::Base_ as an argument.
3. The hook extends _Base_ with the _ActiveRecord::Validations::ClassMethods_ module.

As a result, _Base_ gets both instance methods (`#valid?`) and class methods (`#validates_length_of`)

## The Problem of Chained Inclusions

What happens when you include a module that includes another module?

```ruby
module SecondLevelModule
  def self.included(base)
    base.extend ClassMethods
  end

  def second_level_instance_method; 'ok'; end

  module ClassMethods
    def second_level_class_method; 'ok'; end
  end
end

module FirstLevelModule
  def self.included(base)
    base.extend ClassMethods
  end

  def first_level_instance_method; 'ok'; end

  module ClassMethods
    def first_level_class_method; 'ok'; end
  end

  include SecondLevelModule
end

class BaseClass
  include FirstLevelModule
end
```
_BaseClass_ inlcudes _FirstLevelModule_, which in turn includes _SecondLevelModule_, so you can call both modules' instance methods on an instance of _BaseClass_.

```ruby
BaseClass.new.first_level_instance_method   # => 'ok'
BaseClass.new.second_level_instance_method  # => 'ok'
```

For the class methods, however:

```ruby
BaseClass.first_level_instance_method    # => 'ok'
BaseClass.second_level_instance_method   # => NoMethodError
```

When Ruby calls _SecondLevelModule#included_, the base parameter is not _BaseClass_, but _FirstLevelModule_. As a result, _SecondLevelModule#ClassMethods_ become class methods on _FirstLevelModule_. _ActiveSupport::Concern_ was meant to solve this.

## ActiveSupport::Concern

```ruby
require 'active_support'

module MyConcern
  extend ActiveSupport::Concern

  def an_instance_method; 'an instance method'; end

  module ClassMethods
    def a_class_method; 'a class method'; end
  end
end

MyClass.new.an_instance_method # => 'an instance method'
MyClass.a_class_method         # => 'a class method'
```

### A Look at Concern's Source Code

Two important methods: `extended` and `append_features`. Here is `extended`:

```ruby
module ActiveSupport
  module Concern
    class MultipleIncludedBlocks < StandardError
      def initialize
        super "Cannot define multiple 'included' blocks for a Concern"
      end
    end

    def self.extended(base)
      base.instance_variable_set(:@_dependencies, [])
    end

    # ...
```

When a module extends _Concern_, it calls the extended Hook Method, and defines an `@_dependencies` (initialized to an empty array) class instance variable on the includer.

#### Module#apppend_features

This is a core Ruby method that gets called whenever you _include_ a module (similar behaivor to module#included). However, there is a big difference between the two:

`included` is a Hook method that is normally empty and exists to be overwritten.

`append_features` is not empty and checks whether the included module is already in the includer's chain of ancestors, and if not, adds it.

If you override `append_features`, you can get some surprising results.  For example,

```ruby
module M
  def self.append_features(base); end
end

Class C
  include M
end

# No module M in the ancestor chain
C.ancestors # => [C, Object, Kernel, BasicObject]
```

#### Concern#append_features

_Concern#append_features_ overrides _Module#append_features_. `append_features` is an instance method on _Concern_, so it becomes a class method on modules that extend _Concern_.

For example, if a module _Validations_ extends _Concern_, then it gains a _Validation.append_features_ class method.


#### Inside Concern#append_features

TODO

#### Concern Wrap-Up

_ActiveSupport::Concern_ is a minimalistic dependency management system wrapped into a single module with just a few lines of code.

The code is complex, but using _Concern_ is easy.

Is it too clever?  That depends on who you ask. There are many success stories using metaprogramming, but there is a dark side to it as well.

