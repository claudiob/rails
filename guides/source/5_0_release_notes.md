Ruby on Rails 5.0 Release Notes
===============================

Highlights in Rails 5.0:

* Explicitly halt callback chains

These release notes cover only the major changes. To learn about other
features, bug fixes, and changes, please refer to the changelogs or check out
the list of commits in the main Rails repository on GitHub.

--------------------------------------------------------------------------------

Upgrading to Rails 5.0
----------------------

If you're upgrading an existing application, it's a great idea to have good test
coverage before going in. You should also first upgrade to Rails 4.2 in case you
haven't and make sure your application still runs as expected before attempting
to upgrade to Rails 5.0. A list of things to watch out for when upgrading is
available in the guide [Upgrading Ruby on
Rails](upgrading_ruby_on_rails.html#upgrading-from-rails-4-2-to-rails-5-0).


Major Features
--------------

### Explicitly halt callback chains

In Rails 4.2, when a 'before' callback returns `false` in ActiveRecord,
ActiveModel and ActiveModel::Validations, then the entire callback chain
is halted. In other words, successive 'before' callbacks are not executed,
and neither is the action wrapped in callbacks.

```ruby
class Post < ActiveRecord::Base
  before_save { |post| puts "Callback 1"; false }
  before_save { |post| puts "Callback 2" }
  after_save  { |post| puts "Callback 3" }
end

# Inside a new Rails 4.2 app
post = Post.new
post.save
# Callback 1
# => false
```

In Rails 5.0, returning `false` in a callback will not have the side effect
of halting the callback chain.

```ruby
# Inside a new Rails 5.0 app
post = Post.new
post.save
# Callback 1
# Callback 2
# Callback 3
# => true
```

When you upgrade an existing app from Rails 4.2 to Rails 5.0, you will still
be able to halt callback chains by returning `false`, but you will receive a
deprecation warning about this upcoming change.

```ruby
# Inside a Rails 4.2 app upgraded to Rails 5.0
post = Post.new
post.save
# Callback 1
# DEPRECATION WARNING: Returning `false` in a callback will not implicitly halt a callback chain in the next release of Rails. To explicitly halt a callback chain, please use `raise ActiveSupport::CallbackAborted` instead.
# => false
```

Once you have replaced any `return false` with `raise ActiveSupport::CallbackAborted`, you can opt into
the new behavior and remove the deprecation warning by adding the following
configuration to your `config/application.rb`:

    config.active_support.halt_callback_chains_on_return_false = false

See [#17227](https://github.com/rails/rails/pull/17227) for more details.



Active Model
--------------

### Deprecations

* Deprecated returning `false` as a way to halt ActiveModel and ActiveModel::Valdiations callback chains. The recommended way is to `raise ActiveSupport::CallbackAborted`. ([Pull Request](https://github.com/rails/rails/pull/17227))


Active Record
--------------

### Deprecations

* Deprecated returning `false` as a way to halt ActiveRecord callback chains. The recommended way is to `raise ActiveSupport::CallbackAborted`. ([Pull Request](https://github.com/rails/rails/pull/17227))

Active Support
--------------

### Notable changes

* New config option `config.active_support.halt_callback_chains_on_return_false` to specify whether ActiveRecord, ActiveModel and ActiveModel::Validations callback chains can be halted by returning `false` in a 'before' callback.  ([Pull Request](https://github.com/rails/rails/pull/17227))
