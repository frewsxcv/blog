---
layout: post
title: Navigating to safe navigation
---

Version 2.3 of Ruby introduced a `&.` operator for invoking methods on potentially `nil` objects, dubbed the safe navigation . The behavior of the operator is identical to ActiveSupport's try!:

```ruby
toast.try!(:append, '🥑')
# or
toast&.append('🥑')
```

In the Rails projects I work on, try! is used in *extensively*, so after learning about the new operator.

One of the differences between try! and &. is the latter is implemented by the interpreter and written in C. Knowing this I was curious about the performance and through together a benchmarking script:

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
end; 0
```

```txt
                     user     system      total        real
nil.try!(:length)  1.700000   0.080000   1.780000 (  1.786330)
'a'.try!(:length)  3.170000   0.130000   3.300000 (  3.312468)
nil&.length        0.380000   0.000000   0.380000 (  0.380821)
'a'&.length        0.820000   0.050000   0.870000 (  0.872184)
```

Wow, `&.` is 4x faster than `try!`!

Once I learned it's faster, I went to procrastination driven regex hell and wrote a one line shell script to automatically convert usages of try! to &.:

```sh
find . \
  -type f \
  \( -name '*.rb' -o -name '*.haml' \) \
  -exec gsed -i 's/\.try!(:\([^,)]\+\))/\&.\1/g' '{}' \; \
  -exec gsed -i 's/\.try!(:\([^,)]\+\)\, *\([^)]\+\)\?)/\&.\1(\2)/g' '{}' \;
```

This is assuming GNU sed, so if you're on Mac you'll need to do:

```sh
brew install gnu-sed
alias sed=`which gsed`
```
