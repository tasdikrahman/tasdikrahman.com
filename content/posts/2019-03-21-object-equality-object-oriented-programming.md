---
title: "Object Comparison"
description: "Object Comparison"
tags: [oop, ruby]
comments: true
share: true
cover_image: '/content/images/2019/03/gabriel-gurrola-609666-unsplash.jpg'
---

When do you say two objects are equal?

Taking example of the below two and having ruby as the language,

### Comparing primitive objects

```ruby
irb(main):001:0> 1 == 1
=> true
irb(main):002:0> 'tasdik' == 'tasdik'
=> true
irb(main):003:0>
```

### Comparing custom objects

But what if you are having a custom class defined which has attributes for itself, you can’t really compare them using
`==` default.

For example.

```ruby
irb(main):001:0> module Money
irb(main):002:1>   class Wallet
irb(main):003:2>     attr_accessor :rupee, :paise
irb(main):004:2>
irb(main):005:2>     def initialize(rupee: 0, paise: 0)
irb(main):006:3>       @rupee = rupee
irb(main):007:3>       @paise = paise
irb(main):008:3>     end
irb(main):009:2>   end
irb(main):010:1> end
=> :initialize
irb(main):011:0>
irb(main):012:0> office_wallet = Money::Wallet.new
=> #<Money::Wallet:0x00007fb4328889c8 @rupee=0, @paise=0>
irb(main):013:0> personal_wallet = Money::Wallet.new
=> #<Money::Wallet:0x00007fb430991290 @rupee=0, @paise=0>
irb(main):014:0>
irb(main):015:0> office_wallet == personal_wallet
=> false
irb(main):016:0>
```

But as we can see that the two objects have the same attributes, which would be rupee and paise having the same values.
They should, logically be the same thing and they should have returned `true` as a result of the `==` comparison

### Object equality in ruby

`==` is a general comparison operator in ruby

At the Object level, `==` returns `true` only if obj and other are the same object. Typically, this method is overridden
in descendant classes to provide class-specific meaning.

This is the most common comparison, and thus the most fundamental place where you (as the author of a class) get to 
decide if two objects are “equal” or not.

The double equals method should implement the general identity algorithm to an object, which usually means that you 
should compare the object attributes and not if they are the same object in memory.

### Equivalence relation

The general contract when we are overriding the == operator in ruby is that it should implement an 
[equivalence relation](https://en.wikipedia.org/wiki/Equivalence_relation).

Which has the following properties.

- Reflexive: For any non-null reference value `x` , `x == x` and must return true .
- Symmetric: For any non-null reference value `x` and `y`, `x == y` and `y == x` must return true .
- Transitive: For any non-null reference value `x`, `y` and `z`, if `x == y` and `y == z` return `true` .Then `x == z`
should return `true` .
- Consistent: For any non-null reference value `x`and `y`, multiple invocations of `x == y` must consistently return 
true or consistently return `false` .
- For any non-null reference value `x`, `x == nil` must return `false`.

If the above rules are violated, your equality operator which you have overridden will behave erratically.

### Overriding eql? and hash methods too

But once, you have overridden the `==` operator, you must also override `eql?` and `hash` methods, the reason being that 
even if you have never seen these methods being called anywhere. The reason is these are the methods the Hash object 
is going to use to compare your object if you’re using in as a Hash key. The thing is, Hashes have to be fast to figure
out if a key is already in there and to be able to do this they just avoid comparing every single object, they just go
by “clustering” objects in groups by using the value returned by your object’s “hash” method and then, once in a 
cluster, they compare the objects themselves using “eql?”.

Then searching for a key in a Hash, they first call “hash” in the key to figure out in which group it would be, then 
they compare the key with all the other keys in the group using the “eql?” method.

```ruby
irb(main):001:0> module Wealth
irb(main):002:1>   class Money
irb(main):003:2>     attr_accessor :rupee, :paise
irb(main):004:2>
irb(main):005:2>     def initialize(rupee: 0, paise: 0)
irb(main):006:3>       @rupee = rupee
irb(main):007:3>       @paise = paise
irb(main):008:3>     end
irb(main):009:2>
irb(main):010:2>     def ==(other)
irb(main):011:3>       if other.nil? || !other.instance_of?(Wealth::Money)
irb(main):012:4>         false
irb(main):013:4>       else
irb(main):014:4>         rupee == other.rupee && paise == other.paise
irb(main):015:4>       end
irb(main):016:3>     end
irb(main):017:2>
irb(main):018:2>     alias eql? ==
irb(main):019:2*
irb(main):020:2*     def hash
irb(main):021:3>       [rupee, paise].hash
irb(main):022:3>     end
irb(main):029:2>   end
irb(main):030:1> end
=> nil
irb(main):025:0> office_wallet = Money::Wallet.new
=> #<Money::Wallet:0x00007f96c411ad20 @rupee=0, @paise=0>
irb(main):026:0> personal_wallet = Money::Wallet.new
=> #<Money::Wallet:0x00007f96c40063d0 @rupee=0, @paise=0>
irb(main):027:0>
irb(main):028:0> office_wallet == personal_wallet
=> true
irb(main):029:0>
```

Not the most elegant solution of a `hash` method, but I hope you get the idea.

### A better hash method?

The requirement is that, two objects

- have to be equal if they have the same hash values
- but two objects having same hash values, may or may not be equal

Which brings one to the question, can we have hash functions which never have collisions?

There have been discussions about whether it is possible for a function to exist, which [will never produce hash 
collisions](https://crypto.stackexchange.com/questions/8765/is-there-a-hash-function-which-has-no-collisions). 
But it’s a hard problem to solve.

So when you are trying to override the equals method in ruby, you should also override the hashand `eql?` method.

### Object equality in JAVA

Similar to ruby, when you are trying to implement equivalence relation in java, you have to override the

- `equals`
- `hasCode`

methods in java

```ruby
public boolean equals(Object object) {
  if (this == object) {
    return true;
  } else if (object == null || **getClass() != object.getClass()**) {
    return false;
  }

  MyClass other = (MyClass) object;
  return this.x == other.x && this.y == other.y;
}
```

One of the most common mistakes which people make while overriding the equals method is that instead of using the 
`Object` class, they use the class, in which they are overriding the equals method itself.

### References

- https://www.harukizaemon.com/blog/2005/12/28/how-to-write-eql-in-ruby/
- https://ruby-doc.org/core-2.5.3/Hash.html#class-Hash-label-Hash+Keys
- https://javarevisited.blogspot.com/2011/02/how-to-write-equals-method-in-java.html
- https://stackoverflow.com/questions/7156955/whats-the-difference-between-equal-eql-and
- https://stackoverflow.com/questions/1931604/whats-the-right-way-to-implement-equality-in-ruby'
- https://mauricio.github.io/2011/05/30/ruby-basics-equality-operators-ruby.html
