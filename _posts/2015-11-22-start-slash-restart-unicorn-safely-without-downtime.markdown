---
layout: post
title: "Start/restart unicorn safely without downtime"
date: 2015-11-22 00:42
comments: true
tags: [ruby, rails]
---
[Unicorn](http://unicorn.bogomips.org/) is a cool application server for Rails. If you have a high traffic website, you would like to restart Unicorn without any downtime, upon a new
code deployment or a Gem update which updates unicorn. I wrote an script, exactly for this purpose after learning signal handling basics of Unicorn. Here's the script. The comments
are self-explanatory and fairly give the idea of what is happening.

```sh
#!/bin/bash

# Check if unicorn is already running
unicorn_pid=$(ps -ef | grep "unicorn_rails master" | grep -v grep | awk '{print $2}')
if [[ ! -z $unicorn_pid ]]; then
  echo "Unicorn is already running. Sending USR2 to it"
  kill -USR2 $unicorn_pid
  # Wait till new master comes up so two instances of workers are there
  while [[ $(ps -ef | grep 'unicorn_rails worker\[0\]' | grep -v grep | wc -l) -ne 2 ]]; do
    echo "Waiting till new master comes up and spawns new workers"
    sleep 1
  done
  # Send WINCH to old master
  echo "Winching ID $unicorn_pid"
  kill -WINCH $unicorn_pid
  # Wait till old workers die
  while [[ $(ps -ef | grep 'unicorn_rails worker\[0\]' | grep -v grep | wc -l) -ne 1 ]]; do
    echo "Waiting till old workers die"
    sleep 1
  done
  echo "Killing ID $unicorn_pid"
  kill -QUIT $unicorn_pid
else
  echo "Start fresh copy of unicorn"
  unicorn_rails -c config/unicorn.rb -D
fi
```

Unicorn provides the facility to reload without losing connected clients. The process can be initiated by sending a `USR2` signal to the unicorn master process. Upon recieving this, unicorn will
spawn a new master process and name the old process as `unicorn master (old)`. The new master process will also spawn it's own new workers. Once you are sure that the new workers have been
spawned, you can send a `WINCH` signal to old unicorn master upon receipt of which, old unicorn will shutdown it's workers gracefully. Gracefully here means that the workers will die once
they are done serving their connected clients. Once the old master has shut down all it's workers, we send a QUIT signal to the old master which then dies peacefully. And now we have the new 
unicorn loaded with new code and/or new binary. The explanation of signal behavior can be found [here](http://unicorn.bogomips.org/SIGNALS.html).
