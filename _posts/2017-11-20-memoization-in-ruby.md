---
published: true
layout: post
title: "Using Memoization in Ruby For Efficient Code"
author: "Sergio"
permalink: /2017/11/20/ruby-memoization
date: '2017-11-20 16:16:01 -0600'
---

Memoization is a technique that greatly improves the speed of your accessor methods. It caches the results of memory- or time-intensive operations, that way the work is only done once. Memoization is used so often in Rails, that it was once included as a built-in module in the Rails source code. The module was eventually ditched in favor of a common memoization pattern that we'll be discussing shortly.

## Basic Memoization

This is a common memoization pattern in Ruby:

```ruby
     class User < ActiveRecord::Base
          def instagram_followers
               @instagram_followers ||= instagram_user.followers
          end
     end
```

In the code above, the `||=` basically translates to:

```ruby
@instagram_followers = @instagram_followers || instagram_user.followers
```

That means the first time `instagram_followers` is called, `@instagram_followers` will be nil/falsy and will be set to `instagram_user.followers` by default. Every method call made after the initial one will then return the cached `@instgram_followers` variable. This is really efficient, because we only have to make a network call once, and the response will be stored inside the `@instagram_followers` variable for the method to access quickly.

## Multi-line Memoization

The solution above was simple enough to fit on a single line. But what if we wanted to test for different values first? Here is the most common way to implement memoization on multiple lines:

```ruby
     class User < ActiveRecord::Base
          def main_address
               @main_address ||= begin
                    maybe_main_address = home_address if prefers_home_address?
                    maybe_main_address = work_address unless maybe_main_address
                    maybe_main_address = addresses.first unless maybe_main_address
               end
          end
     end
```

The `begin..end` creates a clock of code in Ruby that can be treated as a single thing.

## What the nil?

These patterns are great, but they do a good job of hiding a detrimental effect. In the first example, what if the user didn't have an instagram account and the instagram followers API returned `nil`? In the next example, what if the user didn't have any addresses, and the block returned `nil`?

Every time we call the method, the instance variable would be `nil`, thus performing the expensive operation each time despite our memoization efforts.

In order to work around this, we would have to differentiate between `nil` and `undefined`:

```ruby
  class User < ActiveRecord::Base
    def instagram_followers
      return @instagram_followers if defined? @instagram_followers
      @instagram_followers = instagram_user.followers
    end
  end
```

```ruby
  class User < ActiveRecord::Base
    def main_address
      return @main_address if defined? @main_address
      @main_address = begin
        main_address = home_address if prefers_home_address?
        main_address ||= work_address
        main_address ||= addresses.first
    end
  end
```

These appear to be hacky solutions, but they work with `nil`, `false` and everything else.

## Conclusion

These are just a couple of ways to memoize your methods in order to make your applications more speed and memory-efficient. These patterns might make your code a little bit more unsightly, which is why someone created a RubyGem called [Memoist](https://github.com/matthewrudy/memoist) in order to provide a cleaner and more friendly API. But it is always nice to see how patterns work under the hood, and perhaps you can create your own RubyGem!