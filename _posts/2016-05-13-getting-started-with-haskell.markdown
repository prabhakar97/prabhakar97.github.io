---
layout: post
title: "Functional Programming Series - Part 1"
date: 2016-05-13 20:02
comments: true
tags: [functional-programming, haskell, recursion]
---
I have been intrigued by functional programming for past couple of years, but never quite prioritized it bigtime. Recently I came across this amazing talk which inspired me
to go all-in to learn FP.

{% youtube "https://www.youtube.com/watch?v=E8I19uA-wGY" %}

I chose Haskell as my preferred language and started with the book [The Haskell Road to Logic, Maths and Programming](http://www.amazon.com/dp/0954300696). The reason was
primarily because this book induces functional thinking without being too theoretical.

I started solving exercises from the book. This post and a few subsequent ones will document my solutions to some of the exercise problems. These solutions may not be most concise 
or efficient because I am just beginning to learn Haskell.

> Define a method removeFst which removes the first instance of an element from a list

```hs
removeFst :: (Eq a) => [a] -> a -> [a]
removeFst (x:xs) y = if x == y
						then xs
                        else x : removeFst xs y

```

> Define a function which counts number of instances of an element in a list

```hs
countChars :: Char -> String -> Int
countChars y [] = 0
countChars y (x:xs) | y == x = 1 + countChars y xs
                    | otherwise = countChars y xs
```

> Define a function named blowUp that takes an string and creates a new string from it by repeating ith character in i times. For example: abcdef should return abbcccddddeeeeeffffff

```hs
blowUp :: Int -> String -> String
blowUp n [x] = take n [x,x..]
blowUp n (x:xs) = (blowUp n [x]) ++ (blowUp (n+1) xs)
```

> Write a function to sort a list of orderable elements

This is a translation of the famous quick sort algorithm to Haskell.

```hs
sort :: (Ord a) => [a] -> [a]
sort [] = []
sort [x] = [x]
sort (x:xs) = sort greater ++ [x] ++ sort lesser
                where greater = [y | y <- xs, y > x]
                      lesser = [y | y <- xs, y < x ]
```

> Write a function that takes a big string and a small string and tells whether the big string starts with the small string

```hs
startWith :: String -> String -> Bool
startWith (x:xs) [y] = y == x
startWith (x:xs) (y:ys) = y == x && startWith xs ys
```

> Write a function that takes a big string and a small string and tells whether the big string contains the small string as a substring

I use the previously defined startWith function to solve this.

```hs
contains :: String -> String -> Bool
contains (x:xs) [] = True
contains (x:xs) y = startWith (x:xs) y || startWith xs y
```

