## Rails 3.2 Standards & Conventions

### Approach

**Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and [KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.**

* General architecture
* Product and API features
* Implementation specifics

### Documentation

**All classes, modules, and methods must be documented using [YARD](http://yardoc.org/) formatted comments.**

### Services

In an attempt to better isolate concerns, we loosely follow some domain driven development (DDD) principles.
Namely, we have added a `services` directory under `app`.

    :::
    |-root
      |-app
        |-assets
        |-controllers
        |-helpers
        |-mailers
        |-models
        |-services <-----
        |-views

**All use cases that meet the following criteria must be organized as service objects under the `services` directory.**

* A complex task oriented transaction is being performed.
* A call is made to an external service.
* Any OS level interaction is performed.
* Sending email, exporting files, etc...

*NOTE: In a typical Rails application, this logic might be found in the model or controller.*

### Logging

We use `ActiveSupport::TaggedLogging` for all logging. This allows us to add implicit tags to each log message which supports better log parsing.

**All error messages are must be tagged with the class and method that raised the exception.** Here's an example:

    :::ruby
    class Geocoder
      def geocode(location)
        begin
          # logic here...
        rescue Exception => ex
          Rails.logger.tagged("Geocoder", "geocode") { Rails.logger.error(ex) }
        end
      end
    end
