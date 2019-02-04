---
layout: post
title: "Metaprogramming 101 in Ruby"
date: 2014-03-01 23:36
comments: true
tags: [ruby]
---
Metaprogramming is fun! Dynamically modifying a program garners a lot of power to the programmer. In this post I show an example situation which is beautifully solved by metaprogramming.
I am a Ruby dilettante. 

#### The situation
I wanted to develop a plugin framework for a side project of mine. There is a higher level class, which should delegate methods calls on itself to the plugin implementer class.
Although, OOPS has a standard solution for this situation using interfaces but here is the beauty and elegance of metaprogramming in ruby.
The code below is the constructor of the higher level class which checks the passed plugin object for interface's implementation and then defines those methods in itself delegating the actual calls to the passed plugin object.

#### The solution
```ruby
def initialize(provider)
  # Check whether the interface is implemented or not
  interface = [
    :get, :put, :entries, :move, :rm!, :mkdir, :file?, :directory?, :exists?, :makedirs
  ]
  interface.each do |method|
    raise "Plugin is not supported" unless provider.respond_to?(method)
  end
  @provider = provider

  # Define the methods supported by provider in this class too
  interface.each do |method|
    self.class.send(:define_method, method) do |*splat|
      @provider.send(method, *splat)
    end
  end
end
```

Boy, so simple it is with ruby! And yeah, I love splat operator!
