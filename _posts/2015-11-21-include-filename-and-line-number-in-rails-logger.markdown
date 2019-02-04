---
layout: post
title: "Include filename and line number in Rails logger"
date: 2015-11-21 23:50
comments: true
tags: [ruby, rails]
---
Rails logging is straightforward and works out of the box. But, sometimes I feel it would 
be really awesome if the log lines would contain the line number and file name which emit 
the log. If you have experience with log4j in Java, you should be able to recall that we
initialize the logger and pass it a class instance. All our log lines are tagged with the
classname. That's cool for debugging purposes.

How to get that behavior in Rails? There is 
one quick and dirty way. Just prepend to the log message `__LINE__` and `__FILE__` which 
contain the line the code is executing and the file it is in, respectively. 

```ruby
logger.info("[#{__FILE__}:#{__LINE__}] The log message")
```

This just works. But shows the full file path which is kind of overkill because it produces very long log lines. The below one improves it to include just the filename.

```ruby
logger.info("[#{__FILE__.split('/').last}:#{__LINE__}] The log message")
```

Ok, this gets the job done but makes our log statements ugly in the code. And it isn't DRY. If
we try to make it DRY, we can't. Because we can't assign the stuff 
on the left that produces file and line number strings, into a variable, because they will refer to the line where variable is assigned, so the log statement will contain wrong value for
line number.

There is another way that involves ActiveSupport's TaggedLogging. If you use a Tagged logger,
you can associate a tag with every log message. That's cool too, especially for searching 
through certain events in logs. TaggedLogging can be used to add file and line number, but
the usage would be similar to the above example. How about doing something that works
seamlessly behind the scenes and is DRY?

After some research, I wrote a class (ok, I assembled it from here and there), whose instance 
can be assigned to Rails log formatter. Here it is. Just put this class in your main
application module and assign it's instance to rails log formatter. Apart from filename
and line number, I have also edited the log line to have some other stuff displayed in fancy
colors. Also, I don't print filename and line if the filename is logger.rb or starts with
log\_. That's because it would create so much spam due to Rails internal log files or
files from various gems.
Use it and change it to your liking.

```ruby
module ApplicationName

  class QLogFormatter
    HOSTNAME = Socket.gethostname
    SEVERITY_TO_COLOR_MAP = {
      'DEBUG' => '35',
      'INFO' => '32',
      'WARN' => '33',
      'ERROR' => '31',
      'FATAL' => '31',
      'UNKNOWN' => '37'
    }
    def call(severity, time, progname, msg)
      return "" if (msg.blank? || msg.strip.blank?)
      formatted_time = time.strftime("%Y-%m-%d %H:%M:%S.") << time.usec.to_s[0..2]
      color = SEVERITY_TO_COLOR_MAP[severity]
      callee = caller[5].split('/').last
      callee = callee.split(':')[0,2].join(':')
      log_line = "\033[0;37m#{HOSTNAME}@#{formatted_time}\033[0m [\033[01;#{color}m#{severity}\033[0m]"
      unless (callee.start_with?('logger') || callee.start_with?('log_'))
        log_line += "[\033[01;36m#{callee}\033[0m]"
      end
      log_line += " #{msg}\n"
    end
  end

  class Application < Rails::Application
    # All your other configs
    config.log_formatter = QLogFormatter.new
  end
end
```
