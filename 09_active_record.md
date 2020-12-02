# Ch 9 The Design of Active Record

Active Record is what maps Ruby objects to database records.

## A Short Active Record Example

```ruby

require 'active_record'
ActiveRecord::Base.establish_connection :adapter => "sqlite3", :database => "dbfile"

class Duck < ActiveRecord::Base
  validate do
    errors.add(:base, "Illegal duck name.") unless name[0] == 'D'
  end
end
```

Once we configure database, and inherit from ActiveRecord::Base, we can run commands like:

```ruby
my_duck = Duck.new
my_duck.name = "Donald" 
my_duck.valid? # => true
my_duck.save!

duck_from_database = Duck.first
duck_from_database.name # => "Donald"
duck_from_database.delete
```

## How Active Record is Put Together

### The Autoloading Mechanism

```ruby
require 'active_support'
require 'active_model'

module ActiveRecord
  extend ActiveSupport::Autoload

  autoload :Base
  autoload :NoTouching
  autoload :Persistence
  autoload :QueryCache
  autoload :Querying
  autoload :Validations
  # ...
```

_ActiveSupport::Autload_ module defines an _autoload_ method that automatically finds and requires teh source code of a module the first time you use the module's name.

As an example, when you first load _ActiveRecord::Base_, _autoload_ automatically requires the file for active_record/base.rb. 

Active Record, in effect, acts like a smart namespace for all the pieces that make up this library.

### ActiveRecord::Base

```ruby
module ActiveRecord
  class Base
    extend ActiveModel::Naming
    extend ActiveSupport::Benchmarkable
    extend ActiveSupport::DescendantsTracker
    ...

    include Core
    include Persistence
    include NoTouching
  end

  ActiveSupoort.run_load_hooks(:active_record, Base)
end
```

The additional line at the bottom allows some of the modules to run thier own configuration code after they've been autoloaded.

Many of these modules include their own modules.  This is where autoloading pays off. Rather than requiring all the module's source code, we can simply just include the module.






