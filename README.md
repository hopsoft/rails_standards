# Rails 3.2 Development Standards Guide

## Approach

Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and 
[KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.

* General architecture
* Product and API features
* Implementation specifics

## Files

* Spaces not tabs
* tab = 2 spaces
* Unix line endings
* UTF-8 encoding

## Documentation

All classes, modules, and methods must be documented using [YARD](http://yardoc.org/) formatted comments.

## General Guidelines

These guidelines are based on Sandy Metz's programming "rules" which she introduced on 
[Ruby Rogues](http://rubyrogues.com/087-rr-book-clubpractical-object-oriented-design-in-ruby-with-sandi-metz/).

The rules are purposefully aggressive and are designed to give you pause so your app won't run amok.
It's expected that you will break them for pragmatic reasons... **alot**.
*See the note on [YAGNI and KISS](#approach).*

* Classes can be no longer than 100 lines of code. 
* Methods can be no longer than 5 lines of code.
* Methods can take a maximum of 4 parameters.
* Controllers should only instantiate 1 object.
* Views should only have access to 1 instance variable.
* Never directly reference another class/module from within a class. 
  *Such references should be passed in*.

## Models

* Never use dynamic finders. e.g. `find_by_...`
* Be thoughtful about using [callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html) 
and [observers](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html) as they can lead to unwanted coupling.

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
* **Important!** Concerns should be isolated and self contained. 
  They should NOT make assumptions about how the receiver is composed at runtime.
  It's unacceptable for a concern to invoke methods defined in other concerns; however, 
  invoking methods defined in the intended receiver is permissible.
* Complex **multi-step** operations should be implemented as a process. _See below._

## Controllers

Controllers should sanitize params before performing any other logic.
The preferred solution is inspired by this [gist from DHH](https://gist.github.com/1975644).

```
class ExampleController < ActionController::Base
  def create
    Example.create(sanitized_params)
  end
  
  def update
    Example.find(sanitized_params[:id]).update_attributes!(sanitized_params)
  end
 
  protected

  def sanitized_params
    @sanitized_params ||= begin
      sanitized = params.slice(:id, :expected_param, :another_expected_param)
      # more sanitization logic if needed
      sanitized
    end
  end
end
```

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

We use the [Yell gem](https://github.com/rudionrails/yell) for logging. 
Here is an example configuration.

```ruby
# example/config/application.rb 
module Example
  class Application < Rails::Application
    log_levels = [:debug, :info, :warn, :error, :fatal]
    
    # %m : The message to be logged
    # %d : The ISO8601 Timestamp
    # %L : The log level, e.g INFO, WARN
    # %l : The log level (short), e.g. I, W
    # %p : The PID of the process from where the log event occured
    # %t : The Thread ID from where the log event occured
    # %h : The hostname of the machine from where the log event occured
    # %f : The filename from where the log event occured
    # %n : The line number of the file from where the log event occured
    # %F : The filename with path from where the log event occured
    # %M : The method where the log event occured
    log_format = Yell.format( "[%d] [%L] [%h][%p][%t] [%F:%n:%M] %m")

    config.logger = Yell.new do |logger|
      logger.adapter STDOUT, :level => log_levels, :format => log_format 
    end
  end
end
```


## Extensions & Monkey Patches

* Be thoughtful about monkey patching and look for alternative solutions first.
* Use an `initializer` to load extensions & monkey patches.

All extensions & monkey patches should live under an `extensions` directory in `lib`.

```
|-project
  |-app
  |-config
  |-db
  |-lib
    |-extensions <-----
```

Use modules to extend objects or add monkey patches.
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

