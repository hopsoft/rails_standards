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
  # extends ...................................................................
  # includes ..................................................................
  # security (i.e. attr_accessible) ...........................................
  # relationships .............................................................
  # validations ...............................................................
  # callbacks .................................................................
  # scopes ....................................................................
  # additional config .........................................................
  # class methods .............................................................
  # public instance methods ...................................................
  # protected instance methods ................................................
  # private instance methods ..................................................
end
```

*NOTE: The comments listed above should exist in the file to serve as a visual reminder of the format.*

### Model Implementation

Its generally a good idea to isolate different concerns into separate modules.
We recommend using Concerns as outlined in [this blog post](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

```
|-project
  |-app
    |-assets
    |-controllers
    |-helpers
    |-mailers
    |-models
      |-concerns <-----
    |-views
```

#### Guidelines

* CRUD operations that are limited to a single model should be implemented in the model.
  For example, a `full_name` method that concats `first_name` and `last_name`
* CRUD operations that reach beyond this model should be implemented as a Concern.
  For example, a `status` method that needs to look at several other models to calculate.
* Simple non-CRUD operations should be implemented as a Concern.
* Complex **multi-step** operations should be implemented as a process. _See below._

## Processes

A process is defined as a **multi-step** operation which includes any of the following.

* A complex task oriented transaction is being performed.
* A call is made to an external service.
* Any OS level interaction is performed.
* Sending emails, exporting files, etc...

In an attempt to better manage processes, we loosely follow some domain driven development (DDD) principles.
Namely, we have added a `processes` directory under `app` to hold our process implementations.

```
|-project
  |-app
    |-assets
    |-controllers
    |-helpers
    |-mailers
    |-models
    |-processes <-----
    |-views
```

We recommend using a tool like [Hero](https://github.com/hopsoft/hero) to help model these processes.

**Important** *Do not use model or controller callbacks to invoke a process. Instead, invoke processes directly from the controller.*

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

**Use an `initializer` to load extensions.**
