---
layout: post
title: "A Gentle Introduction to Core Erlang: Part 1"
description: "An introduction to the Core Erlang language."
tags: [intro, core erlang, erlang, compiler, program transformation]
comments: true
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

Back when I started my PhD (2.5 years ago), I was assigned to instrument Core Erlang code. It proved to be quite a hard task at the beggining, due to the lack of documentation.Since then, we have used Core Erlang in several other projects. Naturally, many people ask me why we choose to work with Core Erlang rather than working directly with Erlang. 

Moreover, it was very hard to work with Core Erlang at the beggining, due to the lack of documentation available.

In this post I will introduce Core Erlang and explain why we use it in our projects.

# Core Erlang

## Erlang made simpler

Core Erlang is a functional and concurrent programming language. In a nutshell, it is a complete but simpler language than Erlang, since many Erlang constructs are just syntactic sugar from a compiler perspective. In fact, Core Erlang is so simple that we refer to it as _a subset of Erlang_ in many of our works.

Generally, Core Erlang is used as an intermediate language by the [Erlang compiler](http://erlang.org/doc/man/compile.html). 
Therefore, developers are not expected to write Core Erlang code directly. This code is usually generated instead, and any change you want to include can be achieved by managing the resulting structure. All of these features make it very convenient for specific tasks such as program transformation, but impractical for the rest of applications.

## From Erlang to Core Erlang

There are several ways to obtain the Core Erlang translation from an Erlang source file.

### From the command line

If you only want to examine the Core Erlang code generated, this method is the easiest one. Just type the following command in the command line:

```shell_session
erlc +to_core module.erl
```

The `+to_core` option will make the Erlang compiler to generate the `module.core` file instead of `module.beam`.
If you open `module.core`, you should be able to read the Core Erlang code translated from your module.

Alternatively, you can perform this translation from the Erlang shell. This method is similar to the previous one, but I find that this way is more convenient when you have to declare a large set of options:

```shell_session
c('module.erl', [to_core]).
```

The result will be the same as in the previous case.

### From an Erlang program

You can also get the associated Core Erlang code from within an Erlang program using the `compile:file/2` function (you can check [here](http://erlang.org/doc/man/compile.html#file-2) the available options and possible outputs).

```erlang
case compile:file(File, [to_core]) of
    {ok, _, CoreForms} ->
      CoreForms;
    _Other ->
      io:fwrite("Error: Could not compile file.~n", []).
end.
```

In this case, the variable `CoreForms` stores the _forms_ that represent your code, and you can **manipulate this data structure** in order to **produce a modified program** (in the next part we will go into more detail about this).

## An example

Let's consider the well-known factorial example, which can be written in Erlang as:

```erlang
fact(0) -> 1;
fact(N) -> N * fact(N-1).
```

We can obtain the corresponding Core Erlang code by using any of the methods previously mentioned.

```erlang
'fact'/1 =
    fun (_@c0) ->
	case _@c0 of
	  <0> when 'true' ->
	      1
	  <N> when 'true' ->
	      let <_@c1> =
		  call 'erlang':'-'
		      (N, 1)
	      in  let <_@c2> =
		      apply 'fact'/1
			  (_@c1)
		  in  call 'erlang':'*'
			  (N, _@c2)
	end
```

Even though this is a basic example, you can already see why Core Erlang is much simpler. Here are some things that happen when you translate Erlang into Core Erlang:
 * **Pattern matching** is **moved** from anywhere **to case statements**.
 * **Guards** are **added to every clause** in a case statement.
 * **Function calls** (to external modules) and **function applications** (calls to the same module) have a **different syntax**.
 * **Function calls** (including built-in functions) are **fully qualified**.

Besides, a **catch-all clause** (i.e., a clause with a pattern `<X> when 'true'`) is added to **case statements** that do not include one.

# Resources