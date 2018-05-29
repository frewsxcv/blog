---
layout: post
title: Navigating to safe navigation
date: 2018-05-29 09:12:57 -0400
---

In December 2013, Ruby 2.3 was released and introduced a [safe navigation operator] to the language: `&.`. The behavior of the operator is identical to [ActiveSupport’s `try!`][try!]:

```ruby
toast = []
# Both lines below push to the array
toast.try!(:push, '🥑')
toast&.push('🥑')

toast = nil
# Both lines below raise NoMethodError
toast.try!(:push, '🥑')
toast&.push('🥑')
```

In the Rails projects I work on, `try!` is used *extensively*, so this new operator quickly caught my eye.

Curious to see how `&.` is implemented, I [looked up the patch][&. source] and noticed that it’s written entirely in C, unlike `try!` which is [written in Ruby][try! source]. Knowing this, I was curious if there is any performance benefit to migrate to `&.`, so I threw together a benchmark:

```ruby
require "benchmark"
require "active_support/all"

Benchmark.bm(14) do |x|
  count = 10_000_000

  x.report("nil.try!(:length)") do
    count.times { nil.try!(:length) }
  end

  x.report("'a'.try!(:length)") do
    count.times { 'a'.try!(:length) }
  end

  x.report("nil&.length      ") do
    count.times { nil&.length }
  end

  x.report("'a'&.length      ") do
    count.times { 'a'&.length }
  end
end
```

Here are the results on my machine:

```txt
                     user     system      total        real
nil.try!(:length)  1.700000   0.080000   1.780000 (  1.786330)
'a'.try!(:length)  3.170000   0.130000   3.300000 (  3.312468)
nil&.length        0.380000   0.000000   0.380000 (  0.380821)
'a'&.length        0.820000   0.050000   0.870000 (  0.872184)
```

Wow, `&.` is 4x faster than `try!`! Upon learning this, I went to regex hell and  back and wrote a one line shell script to automatically convert usages of `try!` to `&.` in all Ruby/HAML files in the current directory and subdirectories:

```sh
find . \
  -type f \
  \( -name '*.rb' -o -name '*.haml' \) \
  -exec sed -i 's/\.try!(:\([^,)]\+\))/\&.\1/g' '{}' \; \
  -exec sed -i 's/\.try!(:\([^,)]\+\)\, *\([^)]\+\)\?)/\&.\1(\2)/g' '{}' \;
```

*Note: this assumes GNU sed, so if you're on MacOS you'll need to `brew install gnu-sed` and replace `sed` calls with `gsed`.*

Maybe this one-liner will help you migrate too!

Thanks for reading!

[try!]: http://api.rubyonrails.org/v5.1.3/classes/Object.html#method-i-try-21
[safe navigation operator]: https://en.wikipedia.org/wiki/Safe_navigation_operator
[&. source]: https://bugs.ruby-lang.org/projects/ruby-trunk/repository/ruby-git/revisions/a356fe1c3550892902103f66928426ac8279e072/diff
[try! source]: https://github.com/rails/rails/blob/6d4bcd439de4ca87374dd6ad03a43deeb3a5e1f6/activesupport/lib/active_support/core_ext/object/try.rb
