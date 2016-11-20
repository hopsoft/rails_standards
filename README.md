# Rails 5.X Development Standards Guide

## TODOs

Items I need to expand on in this document.

- [ ] Always use a first class model for HABTM relationships
- [ ] Only use nested routes when absolutely necessary
- [ ] Use CoffeeScript
- [ ] Use Turbolinks
- [ ] Wire up unobtrusive JavaScript with data attributes

## Approach

### Do Not Fight the Framework

If you find yourself fighting the framework,
[take some time](https://m.signalvnoise.com/give-it-five-minutes-b8115d6f2361#.oq7mwovre) &
ask yourself how the designers of the framework would solve the problems you're facing.
Getting into the heads of those who've created the framework is time well spent.
It will save you hours of work & frustration.

### KISS & YAGNI

Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and
[KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.

* General architecture
* Product and API features
* Implementation specifics

### SPAs & Rails API

Use [Rails API](http://edgeguides.rubyonrails.org/api_app.html) if you're building a
[Single Page Application](https://en.wikipedia.org/wiki/Single-page_application); otherwise,
stick with Rails conventions.

Make the same efforts to learn & understand [Turbolinks](https://github.com/turbolinks/turbolinks)
as you would any other JavaScript framework.

https://www.youtube.com/watch?v=SWEts0rlezA

## Documentation

Make an effort for code to be [self documenting](http://en.wikipedia.org/wiki/Self-documenting).

* Prefer descriptive names in your code. e.g. `user_count` is a better name than `len`.
* Use [YARD](http://yardoc.org/) formatted comments when code documentation is deemed necessary.
* Avoid in method comments as they are a cue that the method is too complex; instead,
refactor into additional classes/methods that better express intent & purpose.

## General Guidelines

These guidelines are based on [Sandi Metz's](http://sandimetz.com/) programming "rules" which she introduced on
[Ruby Rogues](http://rubyrogues.com/087-rr-book-clubpractical-object-oriented-design-in-ruby-with-sandi-metz/).

The rules are purposefully aggressive and are designed to give you pause so your app won't run amok.
It's expected that you will break them for pragmatic reasons...
just be sure that you aware of the trade-offs.

* Classes can be no longer than 100 lines of code.
* Methods can be no longer than 5 lines of code.
* Methods can take a maximum of 4 parameters.
* Controllers should only instantiate 1 object.
* Views should only have access to 1 instance variable.
* Never directly reference another class/module from within a class.
  *Such references should be passed in*.

*Be thoughtful when applying these rules.
If you find yourself fighting Rails, a more pragmatic approach is likely warranted.*

## Models

* Apply extreme caution when using [callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)
and [observers](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html) as they often lead to undesired coupling.

**All models should be organized using the following format.**

```ruby
class MyModel < ActiveRecord::Base
  # extends ...................................................................
  # class methods .............................................................
  # includes ..................................................................
  # relationships .............................................................
  # validations ...............................................................
  # callbacks .................................................................
  # scopes ....................................................................
  # additional config (i.e. accepts_nested_attribute_for etc...) ..............
  # public instance methods ...................................................
  # protected instance methods ................................................
  # private instance methods ..................................................
end
```

*NOTE: The comments listed above should exist in the file to serve as a visual reminder of the format.*

## Controllers

Controllers should use [strong parameters](http://api.rubyonrails.org/classes/ActionController/StrongParameters.html)
to sanitize params before performing any other logic.

## Concerns

Its generally a good idea to isolate concerns into separate modules.
Use concerns as outlined in [this blog post](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

* Concerns should be created in the `app/COMPONENT/concerns` directory.
* Concerns should use the naming convention `COMPONENT` + `BEHAVIOR`.
  For example, `app/models/concerns/model_has_status.rb` or `app/controllers/concerns/controller_supports_cors.rb`.
* __Important!__ Concerns should be isolated and self contained.
  They should NOT make assumptions about how the receiver is composed at runtime.

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
*This will provide introspection help so you can track things down at runtime.*

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

## Domain Objects

Meaningful projects may warrant the creation of __domain objects__,
which can usually be grouped into categories like:

- policies
- presenters
- services
- etc...

Domain objects should be treated as first class citizens of your Rails application.
As such they should live under `app/DOMAIN`.
For example:

- `app/policies`
- `app/presenters`
- etc...

Always apply Rails-like standards to domain object names.
For example, `app/presenters/user_presenter.rb`

Additional reading on the subject of domain objects.

* [The Secret to Rails OO Design](http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)

### Gems & Engines

Sometimes concerns should be grouped into isolated libraries.
This creates clear separation & allows strict control of depedencies.

_Note: This approach does not require you to open-source parts of the app._

Bundler supports 2 strategies that facilitate this type of application structure.

1. [Locally pathed gems](http://bundler.io/v1.7/gemfile.html) - look for the `:path` option
2. [Git hosted gems](http://bundler.io/v1.7/git.html)

_You can also host a [local Gemserver](http://guides.rubygems.org/run-your-own-gem-server/)._

Locally pathed gems & engines should be created under the vendor directory within your project.

```
|-project
  |-vendor
    |-assets
    |-engines <-----
    |-gems    <-----
```

It can be daunting to know when creating a gem or engine is appropriate.
Stephan Hagemann's book on [Component Based Rails Applications](https://leanpub.com/cbra) provides some guidance.

Additional reading on creating modular Rails applications.

* [How to design for loosely-coupled, highly-cohesive components within a Rails application](http://pivotallabs.com/unbuilt-rails-dependencies-how-to-design-for-loosely-coupled-highly-cohesive-components-within-a-rails-application/)
* [Migrating from a single Rails app to a suite of Rails engines](http://pivotallabs.com/migrating-from-a-single-rails-app-to-a-suite-of-rails-engines/)
* [Rails Engines at TaskRabbit](http://tech.taskrabbit.com/blog/2014/02/11/rails-4-engines/)

Here are some popular Rails engines that illustrate how to properly isolate responsibilities to achieve [modularity](http://en.wikipedia.org/wiki/Modular_programming).

- https://github.com/plataformatec/devise
- https://github.com/spree/spree
- https://github.com/radiant/radiant
- https://github.com/refinery/refinerycms
- https://github.com/seyhunak/twitter-bootstrap-rails

## Refactoring

Good software design often emerges empirically from implementation.
The practice of continually moving toward better design is known as refactoring.
Plan on a persistent effort to combat code's natural state of entropy.
_Use prudence to ensure you don't attempt refactoring too much at once._

### Refactoring Priorities

1. Refactor to __methods before classes__
1. Refactor to __classes before libraries__
1. Refactor to __libraries before services__
1. Refactor to __libraries and/or services before a rewrite__

Do not move to services too early.
If you haven't refactored to better abstractions first, services may actually compound your problems.

## Gem Dependencies

Gem dependencies should be hardened before deploying the application to production.
This will ensure application stability.

Use [precise version specifiers](http://gembundler.com/v1.2/gemfile.html).
When using tilde specifiers, be sure to include at least the major & minor numbers.

```ruby
# Gemfile
gem 'rails', '5.0.0.1' # GOOD: exact
gem 'pg', '~>0.19.0'   # GOOD: tilde
gem 'puma', '>=3.6'    # BAD: unspecific
gem 'nokogiri'         # BAD: unversioned
```

Bundler's Gemfile.lock solves the same problem, but inadvertent `bundle updates` can still cause problems.
Better to be explicit in the Gemfile and guarantee stability.

**This will cause your project to slowy drift from the bleeding edge.**
A strategy should be employed to ensure the project doesn't slip too far from contemporary gem versions.
For example, upgrade gems on a regular schedule *(every 3-4 months)* and be vigilant about security patches.
Using `bundle outdated` will help with this.

## Code Quality Tools

It's a good idea to run regular code analysis against your project.
Here are some of the tools we use.

* [Rails Best Practices](https://github.com/railsbp/rails_best_practices)
* [SimpleCov](https://github.com/colszowka/simplecov)
