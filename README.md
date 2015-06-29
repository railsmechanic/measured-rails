# Measured Rails [![Build Status](https://travis-ci.org/Shopify/measured-rails.svg)](https://travis-ci.org/Shopify/measured-rails) [![Gem Version](https://badge.fury.io/rb/measured-rails.svg)](http://badge.fury.io/rb/measured-rails)

This gem is the Rails integration for the [measured](https://github.com/Shopify/measured) gem.

It provides ActiveRecord adapter for persisting and retrieving measurements with their units, and model validations.

## Installation

Using bundler, add to the Gemfile:

```ruby
gem 'measured-rails'
```

Or stand alone:

    $ gem install measured-rails

## Usage

### ActiveRecord

Columns are expected to have the `_value` and `_unit` suffix, and be `DECIMAL` and `VARCHAR`, and defaults are accepted:

```ruby
class AddWeightAndLengthToThings < ActiveRecord::Migration
  def change
    add_column :things, :minimum_weight_value, :decimal, precision: 10, scale: 2
    add_column :things, :minimum_weight_unit, :string, limit: 12

    add_column :things, :total_length_value, :decimal, precision: 10, scale: 2, default: 0
    add_column :things, :total_length_unit, :string, limit: 12, default: "cm"
  end
end
```

A column can be declared as a measurement with its measurement subclass:

```ruby
class Thing < ActiveRecord::Base
  measured Measured::Weight, :minimum_weight
  measured Measured::Length, :total_length
end
```

There are some simpler methods for predefined types:

```ruby
class Thing < ActiveRecord::Base
  measured_weight :minimum_weight
  measured_length :total_length
end
```

This will allow you to access and assign a measurement object:

```ruby
thing = Thing.new
thing.minimum_weight = Measured::Weight.new(10, "g")
thing.minimum_weight_unit      # "g"
thing.minimum_weight_value    # 10
```

Order of assignment does not matter, and each property can be assigned separately and with mass assignment:

```ruby
params = { total_length_unit: "cm", total_length_value: "3" }
thing = Thing.new(params)
thing.total_length   # #<Measured::Length: 3 cm>
```

### Validations

Validations are available:

```ruby
class Thing < ActiveRecord::Base
  measured_length :total_length

  validates :total_length, measured: true
end
```

This will validate that the unit is defined on the measurement, and that there is a value.

Rather than `true` the validation can accept a hash with the following options:

* `message`: Override the default "is invalid" message.
* `units`: A subset of units available for this measurement. Units must be in existing measurement.
* `greater_than`
* `greater_than_or_equal_to`
* `equal_to`
* `less_than`
* `less_than_or_equal_to`

**Special case** Comparison based validation against 0
```ruby
class Thing < ActiveRecord::Base
  measured_length :non_negative_weight

  validates :non_negative_weight, measured: {greater_than: 0}
end
```

Most of these options replace the `numericality` validator which compares the measurement/method name/proc to the column's value. Validations can also be combined with `presence` validator.

**Note:** Validations are strongly recommended since assigning an invalid unit will cause the measurement to return `nil`, even if there is a value:

```ruby
thing = Thing.new
thing.total_length_value = 1
thing.total_length_unit = "invalid"
thing.total_length  # nil

```


## Tests

```
$ bundle exec rake test
```

## Contributing

1. Fork it ( https://github.com/Shopify/measured-rails/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Authors

* [Kevin McPhillips](https://github.com/kmcphillips) at [Shopify](http://shopify.com/careers)
* [Sai Warang](https://github.com/cyprusad) at [Shopify](http://shopify.com/careers)
