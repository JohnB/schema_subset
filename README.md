schema_subset
=============
Nothing beats production data when trying to experience what your users feel.
But when your production database gets so large that it won't _fit_ on your
development machine (let alone copy very quickly) you can get stuck with either
an empty database or fake, unsatisfying data. But if you have enough tables,
with complex relationships between them all, then extracting a meaningful
subset of your production data is nearly impossible. And if your database 
has foreign key constraints that force a copying order, then its even tougher.
This is the problem that schema_subset is meant to solve.

The Plan
========
* User specifies a "main" table, and what percentage of the table to extract
* Understand the schema relationships to find the independent tables
* Iterate to find the tables that are only dependent on the truly independent tables
* Continue iterating to create an entire dependency graph
* Create extraction queries to copy the subset to a parallel set of tables

Example
=======

```ruby
class Car < ActiveRecord::Base
  has_many :drivers
  has_one :owner
  has_one :car_type
end

class Driver < ActiveRecord::Base
  has_many :cars, :through => :car_drivers
end

class Owner < ActiveRecord::Base
  has_many :cars, :through => :car_owners
end

class CarOwner < ActiveRecord::Base
  has_one :car
  has_one :driver
end

class CarDriver < ActiveRecord::Base
  has_one :car
  has_one :driver
end

class CarType < ActiveRecord::Base
end
```

So if we want to extract a half of a percent of of the cars, along with everything that they connect to,
we must:
* select out the IDs of the one percent of cars
* find the IDs from all directly-related tables
* repeat until all tables are accounted for
* reverse the process when creating queries to copy the data

Usage
=====
In your Gemfile:
```ruby
gem 'schema_subset'
```

Set up schema_subset_config.yml:
```ruby
```

In a rakefile:
```ruby
ss = SchemaSubset.new('schema_subset_config.yml')
puts ss.copy_queries
# NOTE: destination tables will *not* have any foreign keys.
#
# SELECT * FROM cars INTO subset.cars WHERE (id % 200) == 1;
# SELECT car_type.* FROM car_type, subset.cars INTO subset.car_type WHERE car_type.id = cars.car_type_id;
# SELECT * FROM drivers INTO subset.drivers WHERE drivers. 
# ...
```


