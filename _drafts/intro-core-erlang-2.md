---
layout: post
title: "A Gentle Introduction to Core Erlang: Part 2"
description: "An introduction to the Core Erlang language."
tags: [intro, core erlang, erlang, BEAM, compiler, program transformation]
comments: true
image:
  feature: abstract-1.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

In the [last post]({{ site.baseurl }}{% post_url 2017-09-15-intro-core-erlang-1 %}),
we learnt the basics of Core Erlang, the intermediate language used by the Erlang compiler.
In this part, we will explain how to write Erlang programs in order to transform Core Erlang programs.

# Core Erlang Program Transformation



## Abstract Syntax Trees

As it is usual in programming languages, Core Erlang programs are represented with an Abstract Syntax Tree (AST for short). 

<figure>
  <img src="/images/intro-core-erlang-1/core-erlang-ast.png" alt="">
  <figcaption></figcaption>
</figure>

## Data Types

Since some constructs in Erlang can be considered syntactic sugar, these can be translated into simpler ones when compiled to Core Erlang. Thus, there are only a few Core Erlang AST data types. The main ones (and what they are used for) are:
 * **'apply'** for function applications.
 * **call** for function calls.
 * **'case'** for case statements.
 * **clause** for clauses in case and receive statements.
 * **cons** for list constructors.
 * **'fun'** for function definitions.
 * **let** for let expressions.
 * **literal** for atoms, numbers, characters...
 * **module** for module definitions.
 * **'receive'** for receive statements.
 * **tuple** for tuples.
 * **var** for variables.

Note that this is not a complete list of the Core Erlang AST data types. However, I consider these to be the most important ones, although the missing types (**c_binary**, **c_catch**, ...) could also appear in your programs (i.e., it depends on your problem).

The type of a `Node` can be obtained with the `cerl:type/1` function. Another option is to match with the record type of `Node` directly (`#c_apply{...}`, `#c_call{...}`, ...), but I prefer the former option because I think it is more readable in general.

## Metaprogramming in Erlang



### Read

```erlang
case compile:file(File, [to_core]) of
    {ok, _, CoreForms} ->
      CoreForms;
    _Other ->
      io:fwrite("Error: Could not compile file.~n", []).
end.
```

### Manipulate
### Write

## Warnings

### Constructor and tuple skeletons.
### Annotations

## Other resources

 * Erlang docs (cerl and others)
 * [Salvador Tamarit - Metaprogramming for Erlang Abstract Format & Core]()
 * merl
 * smerl
