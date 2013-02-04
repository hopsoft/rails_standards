# Rails 3.2 开发标准指南

## Approach

## 使用途径

Apply the [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) and 
[KISS](http://en.wikipedia.org/wiki/KISS_principle) principles to all of the following.

在以下实践中贯彻 [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it) 和
[KISS](http://en.wikipedia.org/wiki/KISS_principle) 原则

* General architecture
* Product and API features
* Implementation specifics

* 一般架构
* 产品及API功能
* 实现细节


## Files

## 文件

* Spaces not tabs
* tab = 2 spaces
* Unix line endings
* UTF-8 encoding

* 采用空格作为缩进而不是制表符
* 采用2个空格作为一个缩进
* 采用Unix行尾结束符
* 采用UTF-8 编码格式

## Documentation

## 文档

Make an effort for code to be [self documenting](http://en.wikipedia.org/wiki/Self-documenting).

* Prefer descriptive names in your code. e.g. `user_count` is a better name than `len`.
* Use [YARD](http://yardoc.org/) formatted comments when code documentation is deemed necessary.
* Avoid in method comments as they are a cue that the method is too complex; refactor instead.

尽量做到代码的[自我解释](http://en.wikipedia.org/wiki/Self-documenting)。

* 优先使用具有描述性的名称。比如:`user_count` 就要优于 `len`。
* 如果确实需要文档化，请使用基于[YARD](http://yardoc.org/)的注释格式。
* 请避免因为方法太过复杂，而需要用注释进行提示的情况。如果有，请重构它。

## General Guidelines

## 一般准则

These guidelines are based on [Sandi Metz's](http://sandimetz.com/) programming "rules" which she introduced on 
[Ruby Rogues](http://rubyrogues.com/087-rr-book-clubpractical-object-oriented-design-in-ruby-with-sandi-metz/).

以下的准则均基于[Sandi Metz's](http://sandimetz.com/)在 [Ruby Rogues](http://rubyrogues.com/087-rr-book-clubpractical-object-oriented-design-in-ruby-with-sandi-metz/) 上所介绍的编程“规则”.

The rules are purposefully aggressive and are designed to give you pause so your app won't run amok.
It's expected that you will break them for pragmatic reasons... **alot**.
*See the note on [YAGNI and KISS](#approach).*

这些规则是故意设计成具有强制性，就是为了通过适度的“中止”，以使你的应用不会失去控制。
希望你是在基于**很多**务实的原因才会打破它。*参考[YAGNI and KISS](#approach)*

* Classes can be no longer than 100 lines of code. 
* Methods can be no longer than 5 lines of code.
* Methods can take a maximum of 4 parameters.
* Controllers should only instantiate 1 object.
* Views should only have access to 1 instance variable.
* Never directly reference another class/module from within a class. 
  *Such references should be passed in*.

* 类中的代码不超过100行
* 方法中的代码不超过5行
* 方法最多采用4个参数
* 控制器应该只实例化一个对象
* 视图应该只获取一个实例变量
* 永远不要直接在类中直接引用其他的类或者模块 *而应该传递这些引用*

*Be thoughtful when applying these rules.
If you find yourself fighting the framework, it's time to be a little more pragmatic.*

*请谨慎的实践以上的规则。如果发现自己过于纠结于代码的结构，也许就是该务实的时候了。*

## Models

## 模型

* Never use dynamic finders. e.g. `find_by_...`
* Be thoughtful about using [callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html) and [observers](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html) as they can lead to unwanted coupling.

* 永远不要使用动态的查找方法。 例如：`find_by_...`
* 仔细斟酌对[回调](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)和[观察者](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html)的使用。他们会导致不必要的耦合。

**All models should be organized using the following format.**

**所有的模型请采用以下的格式组织代码**

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

*注意：以上所列出的注释需明确的编写在文件中，用于在视觉上给出必要的格式提示。*

### Model Implementation

### 模型的实现

Its generally a good idea to isolate different concerns into separate modules.
We recommend using Concerns as outlined in [this blog post](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

针对不同的关注点进行模块分离通常是一个好主意。我们推荐从[这篇博客](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns)中的介绍开始。

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

#### 指导方针

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

* 限于单个模型的CRUD操作应该在模型内实现。比如，采用`full_name`方法合并`first_name`和`last_name`。
* 超出当前模型的CRUD操作应该作为一个关注点。比如，`status`方法就需要依据许多其他的模型进行计算。
* 简单的非CRUD操作应该作为一个关注点实现。
* **非常重要！** 关注点应该独立而且自包含。他们**不**应该假设接收者在运行时是如何组成的。一个关注去调用其他关注中的方法是不被接受的。然而，调用定义在预定接收者中的方法是允许的。
* 复杂的 **多步** 操作应该作为过程实现。_见下文_

## Controllers

## 控制器

Controllers should sanitize params before performing any other logic.
The preferred solution is inspired by [this gist from DHH](https://gist.github.com/1975644).

Here's an example of param sanitization.

实现其他逻辑前，请先对参数进行清理。首选的解决方案灵感来自[DHH的这段gist](https://gist.github.com/1975644)。
下面是一段参数清理的示例。


```ruby
class ExampleController < ActionController::Base
  def create
    Example.create(sanitized_params)
  end

  def update
    Example.find(params[:id]).update_attributes!(sanitized_params)
  end

  protected

  def sanitized_params
    params[:example].slice(:expected_param, :another_expected_param)
  end
end
```

## Processes

## 过程

A process is defined as a **multi-step** operation which includes any of the following.

过程是对“多步”操作的定义，它包含以下内容。

* A complex task oriented transaction is being performed.
* A call is made to an external service.
* Any OS level interaction is performed.
* Sending emails, exporting files, etc...

* 需要执行复杂的、任务导向的事务。
* 对外部服务的调用。
* 执行任何操作系统级别的交互。
* 发送邮件、导出文件，等等...。

In an attempt to better manage processes, we loosely follow some domain driven development (DDD) principles.
Namely, we have added a `processes` directory under `app` to hold our process implementations.

为了更好的管理流程，我们自由（loosely）遵循一些领域驱动模型开发原则。即，我们在`app`目录下添加`processes`文件夹，用于支持（hold）我们对过程的实现。

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

我们推荐使用类似[Hero](https://github.com/hopsoft/hero)的工具来帮助我们对这些过程建模。

**Important** *Do not use model or controller callbacks to invoke a process. Instead, invoke processes directly from the controller.*

**重要** *不要通过模型或者控制器回调去调用过程。而应该取而代之的直接在控制器中调用。*

## Logging

## 日志

We use the [Yell gem](https://github.com/rudionrails/yell) for logging. 
Here is an example configuration.

我们使用 [Yell gem](https://github.com/rudionrails/yell) 进行日志记录。下面是一份配置范例。

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

## 扩展和猴子补丁

* Be thoughtful about monkey patching and look for alternative solutions first.
* Use an `initializer` to load extensions & monkey patches.

* 仔细斟酌是否需要使用猴子补丁，首先应该考虑是否有能够替代的解决方案。
* 使用初始化程序（`initializer`）载入扩展和猴子补丁。

All extensions & monkey patches should live under an `extensions` directory in `lib`.

所有的扩展和猴子补丁都应该存放在`lib`目录下的`extensions`文件夹中。

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

采用模块的方式扩展对象或添加猴子补丁。*这能够在你跟踪异常的时候为你提供一些自省的帮助*

Here's an example:

下面是一个例子：

```ruby
module CowboyString
  def downcase
    self.upcase
  end
end
::String.send(:include, CowboyString)
String.ancestors # => [String, CowboyString, Enumerable, Comparable, Object, Kernel]
```

## Gem Dependencies

## Gem依赖性

Gem dependencies should be hardened before deploying the application to production.
This will ensure application stability.

应用所依赖的Gem版本在发布到生产环境的时候，必须固定。这样可以保证应用的可靠性。

We recommend using [exact or tilde version specifiers](http://gembundler.com/v1.2/gemfile.html).
When using tilde specifiers, be sure to include at least the major &amp; minor numbers.
Here's an example.

我们推荐使用[精确或波浪线版本的说明符](http://gembundler.com/v1.2/gemfile.html)。当使用波浪线说明符时，请确保至少包含了主要和次级版本数字。下面是一个例子。

```ruby
# Gemfile
gem 'rails', '3.2.11' # GOOD: exact
gem 'pg', '~>0.9'     # GOOD: tilde
gem 'yell', '>=1.2'   # BAD: unspecific
gem 'nokogiri'        # BAD: unversioned
```

Bundler's Gemfile.lock solves the same problem; but we discovered that inadvertent `bundle update`s were causing problems.
It's much better to be explicit in the Gemfile and guarantee application stability.

*Bundler所使用的Gemfile.lock文件也可以用来解决这个问题；但是，我们发现无意的`bundle update` 同样会导致问题。因此，为了保证应用的可靠性，在Gemfile中进行精确的定义还是最好的办法。

**This will cause your project to slowy drift away from the bleeding edge.**
A strategy should be employed to ensure your project doesn't drift too far from contemporary gem versions.
For example, upgrade gems on a regular schedule *(every 3-4 months)* and be vigilant about security patches.
Services like [Gemnasium](https://gemnasium.com/) can help with this.

但是这可能会导致你的项目逐渐偏离前沿版本。
你必须采取一定的策略才能保证你不会偏得太远。
比如，周期性*（每3-4个月）*的对gems进行升级，并对安全补丁保持高度警惕。
类似[Gemnasium](https://gemnasium.com/)的服务在这方面会给予帮助。

## A Note on Client Side Frameworks

## 关于客户端框架的注意事项

Exciting things are happening in the world of client side frameworks.

客户端框架的世界不断的有惊喜出现，它们包括：

* [Backbone](http://backbonejs.org/)
* [Ember](http://emberjs.com/)
* [Angular](http://angularjs.org/)
* [Knockout](http://knockoutjs.com/)
* 以及其他种种...

**Be thoughtful about the decision to use a client side framework.**
Ask yourself if the complexity of maintaining 2 independent full stacks is the right decision.
Often times there are better and simpler solutions.

**谨慎选择你所要使用的客户端框架。** 
请从自身考虑，维护2套独立完整的栈（stacks）是否是一个正确的决定。特别是通常都会有更好且更简单的解决方案。

Read the following articles before deciding.
In the end, you should be able to articulate why your decision is the right one.

在做出决定前，请先阅读以下的文章。最后,你应该能够说清楚为什么你的决定是正确的。

* [How Basecamp Next got to be so damn fast without using much client-side UI](http://37signals.com/svn/posts/3112-how-basecamp-next-got-to-be-so-damn-fast-without-using-much-client-side-ui)
* [Rails in Realtime](http://layervault.tumblr.com/post/30932219739/rails-in-realtime)
* [Rails in Realtime, Part 2](http://layervault.tumblr.com/post/31462727280/rails-in-realtime-part-2)

*In either case be mindful of "layout thrashing"
as [described here](http://kellegous.com/j/2013/01/26/layout-performance/).*

*在以上的任何一种情况下都要留心“布局抖动”，可以参考[这篇文章](http://kellegous.com/j/2013/01/26/layout-performance/).中的描述。*
