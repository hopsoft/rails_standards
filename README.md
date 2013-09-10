# Rails 4.0 Development Standards Guide

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

### Model Implementation

Its generally a good idea to isolate different concerns into separate modules.
Use concerns as outlined in [this blog post](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

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

#### Concerns Guide

* CRUD operations that are limited to a single model should be implemented in the model.
  For example, a `full_name` method that concats `first_name` and `last_name`
* CRUD operations that reach beyond this model should be implemented as a Concern.
  For example, a `status` method that needs to look at a different model to calculate.
* Simple non-CRUD operations should be implemented as a Concern.
* **Important!** Concerns should be isolated and self contained.
  They should NOT make assumptions about how the receiver is composed at runtime.
  It's unacceptable for a concern to invoke methods defined in other concerns; however,
  invoking methods defined in the intended receiver is permissible.

## Controllers

Controllers should use [strong parameters](http://api.rubyonrails.org/classes/ActionController/StrongParameters.html)
to sanitize params before performing any other logic.

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
Applying the right strategies will ensure that your project is testable and maintainable well into the future.

### The app Directory

Every Rails project contains the following structure under app.

```ruby
|-project
  |-app
    |-assets
    |-controllers
    |-helpers
    |-mailers
    |-models
    |-views
```

Meaningful projects will warrant the creation of domain objects,
which can typically be isolated into categories.
i.e. presenters, processes, etc...
Steve Klabnik discusses this concept in depth on [his blog](http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design).

Domain objects are first class citizens of the project and should be given proper status.
To accomplish this, create directories under app to store related domain objects.

```
|-project
  |-app
    |-assets
    |-controllers
    |-helpers
    |-mailers
    |-models
    |-processes  <-----
    |-presenters <-----
    |-views
```

Apply Rails-like standards to domain objects when appropriate.
For example, apply similar naming strategies. e.g. `app/presenters/users_presenter.rb` etc...

### Gems & Engines

Sometimes concerns should be grouped into isolated libraries.
This creates clear lines of separation & allows strict control over depedencies.
_Bundler supports local gems with relative paths,
so there's no need to setup a gemserver or open-source parts of the app._

This is an advanced technique.
It can be daunting to know when creating a gem or engine is appropriate.
Stephan Hagemann's presentation at Mountain West Ruby provides
[some guidance](http://www.confreaks.com/videos/2350-mwrc2013-component-based-architectures-in-ruby-and-rails).

Additional reading on this strategy.

* [How to design for loosely-coupled, highly-cohesive components within a Rails application](http://pivotallabs.com/unbuilt-rails-dependencies-how-to-design-for-loosely-coupled-highly-cohesive-components-within-a-rails-application/)
* [Migrating from a single Rails app to a suite of Rails engines](http://pivotallabs.com/migrating-from-a-single-rails-app-to-a-suite-of-rails-engines/)

Custom gems & engines should be created under the vendor directory within your project.

```
|-project
  |-vendor
    |-assets
    |-engines <-----
    |-gems    <-----
```

To create a gem or engine simply run the following commands.

```bash
# gem
cd /path/to/project/vendor/gems
bundle gem GEM_NAME

# engine
cd /path/to/project
rails plugin new vendor/engines/ENGINE_NAME --mountable
```

Update your Gemfile to point to the correct gem locations.

```ruby
# /path/to/project/Gemfile
gem GEM_NAME, path: "/path/to/project/gems/GEM_NAME"
gem ENGINE_NAME, path: "/path/to/project/engines/ENGINE_NAME"
```

Sometimes local gems have dependencies on other local gems.
Simply add the dependency to the gemspec like this.

```ruby
# /path/to/project/vendor/gems/GEM_NAME/GEM_NAME.gemspec
Gem::Specification.new do |spec|
  spec.add_dependency "OTHER_LOCAL_GEM_NAME"
end
```

Then update the Gemfile.

```ruby
# /path/to/project/vendor/gems/GEM_NAME/Gemfile
# this is required for testing the gem in isolation
group :test do
  gem 'OTHER_LOCAL_GEM_NAME', :path => '/path/to/project/vendor/gems/OTHER_LOCAL_GEM_NAME'
end
```

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

## A Note on Client Side Frameworks

Exciting things are happening in the world of client side frameworks.

* [Ember](http://emberjs.com/)
* [Angular](http://angularjs.org/)
* [Backbone](http://backbonejs.org/)
* [Knockout](http://knockoutjs.com/)
* and many others...

**Be thoughtful about the decision to use a client side framework.**
Ask yourself if the complexity of maintaining 2 independent full stacks is the right decision.
Often times there are better and simpler solutions.

Read the following articles before deciding.
In the end, you should be able to articulate why your decision is the right one.

* [How Basecamp Next got to be so damn fast without using much client-side UI](http://37signals.com/svn/posts/3112-how-basecamp-next-got-to-be-so-damn-fast-without-using-much-client-side-ui)
* [Rails in Realtime](http://layervault.tumblr.com/post/30932219739/rails-in-realtime)
* [Rails in Realtime, Part 2](http://layervault.tumblr.com/post/31462727280/rails-in-realtime-part-2)

We generally agree with DHH regarding heavy client side frameworks.

<blockquote class="twitter-tweet"><p>JavaScript is like a spice. Best used in sprinkles and moderation.</p>&mdash; DHH (@dhh) <a href="https://twitter.com/dhh/statuses/374656854825005056">September 2, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

We've discovered that thoughtful use of Backbone or Knockout within Rails meets most of our needs.

