---
layout: post
title: Testing Java Code With JRuby
author: Andrew Semprebon
tags: code jruby software java ruby testing minitest
category: Code
---

Here at Mobile System 7, most of our code is Ruby, but we do have a small
amount of java code where ruby deployment would be inappropriate. However,
just because we have to use java in the code  doesn't mean we need to use
java to test the code.

There are a number of good reasons for using Ruby for testing java code

1. Ruby is more concise &mdash; Less code is better.
2. Ruby testing frameworks like MiniTest are more readable &mdash; using actual english to describe tests ratherThanCamelcaseIdentifiers
3. We can use some of Ruby's dynamic debugging tools like pry
4. We have more experience with ruby and ruby test frameworks than with java/junit

As an example, we will develop a java class that summarizes counts of objects in
a collection, returning a Map with unique objects from the collection as keys
and the count as values. For example:

    ArrayList list = java.util.Arrays.asList(
        new String[] {"Mann", "Jones", "Trenberth", "Mann"});
    Counter.count(authors)).get("Mann"); // returns 2

This is basically equivalent to the following ruby code:

    authors = ["Mann", "Jones", "Trenberth", "Mann"]
    authors.inject({}) { |result, item| result.merge(item => (result[item] || 0) + 1) }
    authors["Mann"]  # returns 2

## Setup

At its simplest, you just need to have Ruby installed. The standard Ruby library
includes the MiniTest framework, which provides for several different testing
styles. Since our other ruby tests are written with rspec, we use the MiniTest::Spec
framework.

## Writing the First Test

We start with the ruby test, which looks like this:

    require 'minitest/autorun'
    require "java"

    $CLASSPATH << "classes"
    java_import Java::ComMs7::Counter

    describe Counter do

      let(:counter) { Counter.new }

      describe ".count" do
        it "returns empty hash for empty collection" do
          counter.count([]).must_equal({})
        end
      end
    end

The first few lines load minitest and the jruby/java libraries, adds the
directory where our java class files are stored to Ruby's load path, and
imports our Counter class so we don't have to specify the package each time.
One thing to note here is the way java packages are translated into ruby
modules: the periods are replaced with camel case to form a submodule name
in JRuby's Java module.

Then, we specify the class being tested ("describe Counter") as well as
the method (".count"), and then a test that verifies that the method
returns an empty hash if an empty array is passed in. By starting with
aa simple test like this, we can verify that all our setup is working
correctly.

## Writing the Counter java class

Running this just results in a class not found error, so lets create
the java class with just enough implementation to get the test to pass:

    package com.ms7;

    import java.util.Map;
    import java.util.HashMap;

    public class Counter {

      public Map<Object, Integer> count(Iterable<Object> items) {
        return new HashMap<Object, Integer>();
      }
    }

For now, this just returns an empty hash. When we run this, we get:

    $ ruby test/counter_test.rb
    Run options: --seed 9924

    # Running tests:

    .

    Finished tests in 0.018000s, 55.5556 tests/s, 55.5556 assertions/s.

    1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
    $

This tells us that the test passed.

## Another Test

Lets add a more substantial test:

        it "returns count of 2 when given collection of 2 like items" do
          collection = ["Mann", "Mann"]
          counter.count(collection).must_equal({ "Mann" => 2 })
        end

Here, we create an array of two identical strings, and expect a Map (hash)
with a count of 2. As expected, the test fails:

    test_0001_returns hash with count(Java::ComMs7::Counter::count::with collection of 2 like items) [test/counter_test.rb:25]:
    Expected: {"Mann"=>2}
      Actual: {}

## Actually Computing the Count and Things Go Awry

Now we modify the count method to actually count things:

      public Map<Object, Integer> count(Iterable<Object> items) {
        HashMap<Object, Integer> result = new HashMap<Object, Integer>();
        for(Object item: items) {
          int count = (result.containsKey(item)) ? result.get(item) + 1 : 1;
          result.put(item, count);
        }
        return result;
      }

Looks good, right? Unfortunately, when we run the test, again. it still fails:

    No visible difference in the Hash#inspect output.
    You should look at the implementation of #== on Hash or its members.
    {"Mann"=>2}

Well, that's odd. It looks the same but isn't actually equal.

## Using Pry

To figure this out, we are going to use pry, a ruby debugging utility.
First, we install it using the ruby gems package manager:

    gem install pry

And then modify our test:

    require "pry"
    ...
        it "returns count of 2 when given collection of 2 like items" do
          collection = ["Mann", "Mann"]
          binding.pry
          counter.count(collection).must_equal({ "Mann" => 2 })
        end

The idea here is that when we get to that "binding.pry" line, we will be
dropped into an interactive ruby session where we can poke at the values
of things:

    From: /Users/semprebon/Library/JRuby/jruby-1.7.2/lib/ruby/gems/shared/gems/minitest-4.5.0/lib/minitest/unit.rb @ line 1318 Java::ComMs7::Counter::count::with collection of 2 like items#test_0001_returns hash with count:
    ...
    [1] pry(#<Java::ComMs7::Counter::count>)>

Ok, lets poke around:

    [1] pry(#<Java::ComMs7::Counter::count>)> collection
    => ["Mann", "Mann"]
    [2] pry(#<Java::ComMs7::Counter::count>)> counter.count(collection)
    => {"Mann"=>2}

So far, everything looks good. Lets try comparing the result with what we expect:

    [3] pry(#<Java::ComMs7::Counter::count>)> counter.count(collection) == {"Mann"=>2}
    => false

Ah, yeah, that should be true. Maybe we are comparing different classes?

    [4] pry(#<Java::ComMs7::Counter::count>)> counter.count(collection).class
    => Java::JavaUtil::HashMap

Yeah, that's the problem. The java class is returning a java HashMap object,
but we are comparing it to a ruby Hash object. In any case, we can convert the java HashMap
into a ruby Hash with the to_hash method. Lets see if that works:

    [5] pry(#<Java::ComMs7::Counter::count>)> counter.count(collection).to_hash == {"Mann"=>2}
    => true

Great, we exit out of pry and modify the test code:

        it "returns count of 2 when given collection of 2 like items" do
          collection = ["Mann", "Mann"]
          counter.count(collection).to_hash.must_equal({ "Mann" => 2 })
        end

And sure enough, the test now passes!

## Test Data Generation

Finally, an example of how to generate test data using jRuby. Lets add a
more complex test that has a random assortment of different items in the
collection. We create a known quantity of different items, then shuffle
them to create a random order:

    it "returns correct count with mix of items" do
      collection = ([true]*97 + [false]*3).shuffle
      counter.count(collection).to_hash.must_equal({ true => 97, false => 3 })
    end

As expected, this also passes.

## Summary

This should give you some idea of how to test your java using ruby, and the
advantages of doing so. Here are some reference links to get you started:

* [A MiniTest::Spec Turorial](http://www.rubyinside.com/a-minitestspec-tutorial-elegant-spec-style-testing-that-comes-with-ruby-5354.html)
* [MiniTest Documentation](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/minitest/unit/rdoc/MiniTest.html)
* [Calling Java From JRuby](https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby)
* [Pry](https://github.com/nixme/pry-debugger)

