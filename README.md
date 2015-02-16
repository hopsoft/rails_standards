# Rails 4.X Development Standards Guide

## Approach

Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and
[KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.

* General architecture
* Product and API features
* Implementation specifics

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
If you find yourself fighting Rails too often, a more pragmatic approach might be warranted.*

## Models

* Never use dynamic finders. e.g. `find_by_...`
* Apply extreme caution when using [callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)
and [observers](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html) as they typically lead to unwanted coupling.

**All models should be organized using the following format.**

```ruby
class MyModel < ActiveRecord::Base
  # extends ...................................................................
  # includes ..................................................................
  # relationships .............................................................
  # validations ...............................................................
  # callbacks .................................................................
  # scopes ....................................................................
  # additional config (i.e. accepts_nested_attribute_for etc...) ..............
  # class methods .............................................................
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
* CRUD operations that are limited to a single model should be implemented in the model.
  For example, a `full_name` method that concats `first_name` and `last_name`
* CRUD operations that reach beyond this model should be implemented as a Concern.
  For example, a `status` method that needs to look at a different model to calculate.
* Simple non-CRUD operations should be implemented as a Concern.
* __Important!__ Concerns should be isolated and self contained.
  They should NOT make assumptions about how the receiver is composed at runtime.
  It's unacceptable for a concern to invoke methods defined in other concerns; however,
  invoking methods defined in the intended receiver is permissible.

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

## Project Structure

Unfortunately, some of the defaults in Rails seem to encourage monolithic design.
This is especially true for developers new to the framework.
However, it's important to note that Ruby & Rails include everything you need to create well organized projects.

The key is to keep concerns physically isolated.
Applying the right strategies will ensure your project is testable and maintainable well into the future.

### Domain Objects

Meaningful projects warrant the creation of __domain objects__,
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

Bundler supports 2 strategies that support this type of application structure.

1. [Locally pathed gems](http://bundler.io/v1.7/gemfile.html) - look for the `:path` option
2. [Git hosted gems](http://bundler.io/v1.7/git.html)

_You can also host a [local Gemserver](http://guides.rubygems.org/run-your-own-gem-server/)._

It can be daunting to know when creating a gem or engine is appropriate.
Stephan Hagemann's presentation at Mountain West Ruby provides
[some guidance](http://www.confreaks.com/videos/2350-mwrc2013-component-based-architectures-in-ruby-and-rails).
He is also writing a book on [Component based Rails Applications](https://leanpub.com/cbra).

Custom gems & engines should be created under the vendor directory within your project.

```
|-project
  |-vendor
    |-assets
    |-engines <-----
    |-gems    <-----
```

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

## Refactoring to Better Design

Good software design often emerges empirically from implementation.
The practice of continually moving toward better design is known as refactoring.

Plan on a persistent effort to combat code's natural state of entropy.
_Use prudence to ensure you don't attempt refactoring too much at once._

### Refactoring Priorities

1. Refactor to __methods before classes__
1. Refactor to __classes before libraries__
1. Refactor to __libraries before services__
1. Refactor to __libraries and/or services before a rewrite__

## Gem Dependencies

Gem dependencies should be hardened before deploying the application to production.
This will ensure application stability.

We recommend using [exact or tilde version specifiers](http://gembundler.com/v1.2/gemfile.html).
When using tilde specifiers, be sure to include at least the major & minor numbers.
Here's an example.

```ruby
# Gemfile
gem 'rails', '3.2.11' # GOOD: exact
gem 'pg', '~>0.9'     # GOOD: tilde
gem 'yell', '>=1.2'   # BAD: unspecific
gem 'nokogiri'        # BAD: unversioned
```

Bundler's Gemfile.lock solves the same problem, but we discovered that inadvertent `bundle updates` can cause problems.
It's much better to be explicit in the Gemfile and guarantee app stability.

**This will cause your project to slowy drift away from the bleeding edge.**
A strategy should be employed to ensure the project doesn't drift too far from contemporary gem versions.
For example, upgrade gems on a regular schedule *(every 3-4 months)* and be vigilant about security patches.
Using `bundle outdated` will help with this.

## Code Quality Tools

It's a good idea to run regular code analysis against your project.
Here are some of the tools we use.

* [Rails Best Practices](https://github.com/railsbp/rails_best_practices)
* [SimpleCov](https://github.com/colszowka/simplecov)

## A Note on Client Side Frameworks

Exciting things are happening in the world of [client side frameworks](http://todomvc.com/).

* [React](http://facebook.github.io/react/)
* [Angular](http://angularjs.org/)
* [Ember](http://emberjs.com/)
* [Backbone](http://backbonejs.org/)
* [Knockout](http://knockoutjs.com/)
* [and many others...](http://todomvc.com/)

**Be thoughtful about the decision to use a client side framework.**
Ask yourself if the complexity of maintaining 2 independent full stacks is the right decision.
Often times there are better and simpler solutions.

Read the following articles before deciding.
In the end, you should be able to articulate why your decision is the right one.

* [How Basecamp Next got to be so damn fast without using much client-side UI](http://37signals.com/svn/posts/3112-how-basecamp-next-got-to-be-so-damn-fast-without-using-much-client-side-ui)
* [Rails in Realtime](http://layervault.tumblr.com/post/30932219739/rails-in-realtime)
* [Rails in Realtime, Part 2](http://layervault.tumblr.com/post/31462727280/rails-in-realtime-part-2)

<blockquote class="twitter-tweet"><p>JavaScript is like a spice. Best used in sprinkles and moderation.</p>&mdash; DHH (@dhh) <a href="https://twitter.com/dhh/statuses/374656854825005056">September 2, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

We generally agree with DHH regarding client side frameworks and have discovered that
thoughtful use of something like [Vue](http://vuejs.org/) meets most of our needs.

