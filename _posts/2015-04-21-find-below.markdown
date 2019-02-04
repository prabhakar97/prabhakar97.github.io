---
layout: post
title: "Find below"
date: 2015-04-21 03:22
comments: true
tags: [shell, linux]
---
This is a small piece of snippet that can go into your `.zshrc` and make your life easier sometimes. This whips up a new command named `findbelow` for you. This command takes as argument, a string and returns all the files below your current directory whose names contain the entered string as a substring.

```sh
findbelow () {
  find ./ -regex ".*/$1.*"
}
```

### Bonus
The below piece of snippet enables you to jump n directories upwards from your current `pwd` by typing `u n`. For example: `u 5` will take you 5 directories up from your current position.
```sh
u () {
  set -A ud
  ud[1+${1-1}]=
  cd ${(j:../:)ud}
}
```
This has been tested on zsh.
