
In Rails, you can include modules and you gain both instance and class methods.  How does that happen?

_ActiveSupport::Concern_ allows you to add both and makes it easy to roll that functionality into other modules.

## Rails Before Concern

_ActiveRecord::Base_ is an assembly of dozens of modules that defien both instance methods and class methods.

Before _ActiveSupport::Concern_ there was another way to roll those methods into _Base_.

## The Include-and-Extend Trick

In Rails 2, all validaiton methods were defined in _ActiveRecord::Validations_ (there was no Active Model at this time).

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

For teh class methods, however:

```ruby
BaseClass.first_level_instance_method    # => 'ok'
BaseClass.second_level_instance_method   # => NoMethodError
```

When Ruby calls _SecondLevelModule#included_, the base parameter is not _BaseClass_, but _FirstLevelModule_. As a result, _SecondLevelModule#ClassMethods_ become class methods on _FirstLevelModule_. _ActiveSupport::Concern_ was meant to solve this.

## ActiveSupport::Concern

