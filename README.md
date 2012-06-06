# Rails 3.2 Development Standards Guide

## Approach

**Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and [KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.**

* General architecture
* Product and API features
* Implementation specifics

## Files

* Spaces not tabs
* tab = 2 spaces
* Unix line endings
* UTF-8 encoding

## Documentation

**All classes, modules, and methods must be documented using [YARD](http://yardoc.org/) formatted comments.**

## Models

**All models should be organized using the following format.**

```ruby
class MyModel < ActiveRecord::Base
  # extends ...
  # includes ...
  # security ... (i.e. attr_accessible)
  # relationships ...
  # validations ...
  # callbacks ...
  # scopes ...
  # class methods ...
  # public instance methods ...
  # protected instance methods ...
  # private instance methods ...
end
```

*NOTE: The comments listed above should exist in the file to serve as a visual reminder of the format.*

## Services

In an attempt to better isolate concerns, we loosely follow some domain driven development (DDD) principles.
Namely, we have added a `services` directory under `app`.

```
|-project
  |-app
    |-assets
    |-controllers
    |-helpers
    |-mailers
    |-models
    |-services <-----
    |-views
```

**All use cases that meet the following criteria must be organized as service objects under the `services` directory.**

* A complex task oriented transaction is being performed.
* A call is made to an external service.
* Any OS level interaction is performed.
* Sending email, exporting files, etc...

*NOTE: In a typical Rails application, this logic might be found in the model or controller.*

## Logging

We use `ActiveSupport::TaggedLogging` for all logging. This allows us to add implicit tags to each log message which supports better log parsing.

**All error messages are must be tagged with the class and method that raised the exception.**

Here's an example:

```ruby
# /project/app/services/geocoder.rb
class MyObject
  def foo
    begin
      # logic here ...
    rescue Exception => ex
      Rails.logger.tagged("MyObject", "foo") { Rails.logger.error(ex) }
    end
  end
end
```

## Extensions & Monkey Patches

**Use modules to extend objects or add monkey patches.**

*This provides some introspection assistance when you need to track down weirdness.*

Here's an example:

```ruby
module CowboyString
  def downcase
    self.upcase
  end
end
::String.send(:include, CowboyString)
String.ancestors # => [String, CowboyString, Enumerable, Comparable, Object, Kernel]
```

**All extensions should live under an `extensions` directory in `lib`.**

```
|-project
  |-app
  |-config
  |-db
  |-lib
    |-extensions <-----
```
