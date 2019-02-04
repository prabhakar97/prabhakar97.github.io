---
layout: post
title: "Configure DatabaseCleaner with Rspec for Mongoid 5 and beyond"
date: 2016-06-26 21:05:22 -0400
comments: true
tags: [rails, ruby, mongodb]
---
[Database Cleaner](https://github.com/DatabaseCleaner/database_cleaner) is a nifty gem for streamlining tests. Configuring it is straightforward
but it didn't work for me out of the box for Mongoid 5.

Here's my Gemfile segment for testing.

```ruby
group :test do
  gem "factory_girl_rails"
  gem "rspec-rails", '~> 3.4'
  gem 'faker'
  gem 'database_cleaner', git: 'git://github.com/DatabaseCleaner/database_cleaner.git'
end
```

I added a `require 'support/database_cleaner'` in my `spec_helper`. And here's my `database_cleaner.rb`.

```ruby
RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.strategy = :truncation
    DatabaseCleaner.clean
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end

end
```

When I ran my spec, I was greeted with an unexpected ugly error.

```text
/home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/mongo2/truncation_mixin.rb:29:in `collections': undefined method `collections' for #<Mongo::Client:0x47021052998580 cluster=127.0.0.1:27017> (NoMethodError)
	from /home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/mongo2/truncation_mixin.rb:9:in `clean'
	from /home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/base.rb:92:in `clean'
	from /home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/configuration.rb:79:in `block in clean'
	from /home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/configuration.rb:79:in `each'
	from /home/prabhakar/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/database_cleaner-f052d64d3be9/lib/database_cleaner/configuration.rb:79:in `clean'
```

After looking at the code in the gem, I finally found out what was wrong in it. I created a
[commit for fix](https://github.com/prabhakar97/database_cleaner/commit/3cf0d1b81e4a118fd173d697f032a9aff4f431de) and submitted a pull request to
the maintainer. Looking at my commit and the relevant files [truncation.rb](https://github.com/DatabaseCleaner/database_cleaner/blob/master/lib/database_cleaner/mongoid/truncation.rb) and [truncation_mixin.rb](https://github.com/DatabaseCleaner/database_cleaner/blob/master/lib/database_cleaner/mongo2/truncation_mixin.rb), you should be able to piece together the problem.

Meanwhile, if you are using Mongoid 5 and aren't able to get it working you could simply point to my fork of database_cleaner by updating your Gemfile
to have: 

```ruby
gem 'database_cleaner', git: 'git://github.com/prabhakar97/database_cleaner.git'
```
