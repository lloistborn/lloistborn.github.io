# Overview
There are a lot of things that need to be clarified about how developers handle errors. Golang programmers, in particular, used to think this is the best way to do it.
```
if err != nil
```
Unfortunately, this is a misconception. It is either the new programmers who never read the documentation or they don't know about the design pattern behind it.
