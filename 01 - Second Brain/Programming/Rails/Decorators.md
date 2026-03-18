# Refs
1. [Simple Decorators with SimpleDelegator](https://dev.to/lightflight/simple-decorators-with-simpledelegator-23nj)
2. [Minimal Decorator Ruby](https://remimercier.com/minimal-decorator-ruby/#fnref:1)
3. [How to implement Decorator pattern in Ruby on Rails?](https://dev.to/vladhilko/series/22841)
4. [Evaluating Alternative Decorator Implementations In Ruby](https://thoughtbot.com/blog/evaluating-alternative-decorator-implementations-in)

# Topics

1. ???
2. 

# Raw Info

Decorator allows adding behavior to an object dynamically without modifying the original class.  
In Rails it's often used for presentation logic, but it can also be used for caching, logging, authorization, metrics, or other cross-cutting concerns.

## How to Create

Let's create some decorators to enhance these models.

```
# app/decorators/post_decorator.rb

class PostDecorator < SimpleDelegator
  STATUS_COLORS = {
    published: :green,
    draft: :indigo
    archived: :gray,
    deleted: :red
  }.freeze

  def status_color = STATUS_COLORS[status.to_sym]
end
```


Let's clean this up adding `method_missing` method to all our models.

```
# app/models/application_record.rb

class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

+  include Decoratable
end
```

```
# app/models/concerns/decoratable.rb

module Decoratable
  extend ActiveSupport::Concern

  def respond_to_missing?(method_name, ...)
    return false unless decorator_class

    decorator_class.instance_methods.include?(method_name)
  end

  def method_missing(method_name, ...)
    return super unless respond_to_missing?(method_name)

    decorated_instance.public_send(method_name, ...)
  end

  private

  def decorator_class
    @decorator_class ||= "#{model_name}Decorator".safe_constantize
  end

  def decorated_instance
    @decorated_instance ||= decorator_class.new(self)
  end
end
```

This will check if the method exists in the corresponding decorator before throwing an error, allowing us to use decorator methods as if they were defined in the models themselves!  

```
- @posts = Post.all.map { PostDecorator.new(it) }
+ @posts = Post.all

- <%= AuthorDecorator.new(post.author).full_name %>
+ <%= post.author.full_name %>
```

This approach gives us the best of both worlds: clean models and convenient access to decorator methods. The `method_missing` implementation acts as a bridge between our models and decorators, making the code more maintainable and easier to work with.

Much cleaner now, right?

## Ruby’s `method_missing` to the rescue

`method_missing` is how Ruby handles method calls made on objects where said methods are not defined. Ruby passes the method call along [the ancestry chain](https://remimercier.com/beginners-introduction-to-ruby-classes-objects/) until it can either resolves it or raises a `NoMethodError`.

SimpleDelegator works by implementing the method_missing method that ruby calls when it can’t find a variable or method name. It then asks the delegate object if it responds to the apparently missing method implementation and if it does it calls it.

## Normalizing the behavior to create other decorators

One way to gather default behavior shared across various decorators is to rely on inheritance. I can create an ApplicationDecorator whose job is to handle instantiation, and forwarding method calls to the underlying record.

Then, I can have my TeacherDecorator inherit from the ApplicationDecorator.

```ruby
class ApplicationDecorator < SimpleDelegator ; end
```

## Realizations

## Plain ruby

Plain Ruby Decorator is just a class that accepts a model object and returns a new object with all desired methods. For example:  

```
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User

    attr_reader :user

    def initialize(user)
      @user = user
    end

    def full_name
      "#{user.name} #{user.surname}"
    end

    def full_address
      "#{user.country} #{user.city} #{user.street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

The interface looks good, but we have to repeat `user` every time inside the decorator. Let's try to change this with our second option - `SimpleDelegator Decorator`

## SimpleDelegator

[SimpleDelegator](https://ruby-doc.org/stdlib-2.5.1/libdoc/delegate/rdoc/SimpleDelegator.html) Decorator is the same as a Plain Ruby Decorator, but with only one difference - we delegate all supported method calls to the object passed into the constructor. Let's take a look at the following example:  

```
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User < SimpleDelegator

    def full_name
      "#{name} #{surname}"
    end

    def full_address
      "#{country} #{city} #{street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

With SimpleDelagator our solution looks much clearer and more elegant. This approach solves 95% of the common cases and I really like it because of its simplicity, but if you need something more complex, here is our last option, `draper` decorator.

## Draper  Decorator

[Draper](https://github.com/drapergem/draper) is the most popular gem to implement Decorator pattern for Rails. With this gem our example would look like this:  

```
# app/units/decorators/user.rb

# frozen_string_literal: true

module Decorators
  class User < Draper::Decorator

    delegate_all

    def full_name
      "#{name} #{surname}"
    end

    def full_address
      "#{country} #{city} #{street}"
    end

  end
end

decorated_user = Decorators::User.new(User.last)
decorated_user.full_name
decorated_user.full_address
```

You can read more about draper [here](https://github.com/drapergem/draper) and decide for yourself whether it makes sense to add it or not.

## Advantages

- Decorator can be easily tested separately from the controller
- Decorator is DRY and reusable
- Decorator allows us to avoid FAT model and stick to SOLID principles
- Decorator reduces chances of breaking existing logic and increases product stability
- Decorator reduces maintenance cost

## [Use it instead of inheritance](https://thoughtbot.com/blog/evaluating-alternative-decorator-implementations-in#use-it-instead-of-inheritance)

An example I’ve seen a couple of times for decorators is the “coffee with milk and sugar” example. One that I found particularly helpful was [Luke Redpath’s article](http://lukeredpath.co.uk/blog/decorator-pattern-with-ruby-in-8-lines.html).

The problems with inheritance include:

- Choices are made statically.
- Clients can’t control how and when to decorate a component.
- Tight coupling.
- Changing the internals of the superclass means all subclasses must change.

## [How a decorator works](https://thoughtbot.com/blog/evaluating-alternative-decorator-implementations-in#how-a-decorator-works)

Using Gang of Four terms, a **decorator** is an object that encloses a**component** object. It also:

- conforms to interface of component so its presence is transparent to clients.
- forwards (delegates) requests to the component.
- performs additional actions before or after forwarding.

This approach is more flexible than inheritance because you can mix and match responsibilities in more combinations and because the transparency lets you nest decorators recursively, it allows for an **unlimited number**of responsibilities.

## [Alternative implementations in Ruby](https://thoughtbot.com/blog/evaluating-alternative-decorator-implementations-in#alternative-implementations-in-ruby)

I’ve researched and found four common implementations of decorators in Ruby:

- Module + Extend + Super decorator
- Plain Old Ruby Object decorator
- Class + Method Missing decorator
- SimpleDelegator + Super + Getobj decorator

There’s probably others but these seem to be the most common.