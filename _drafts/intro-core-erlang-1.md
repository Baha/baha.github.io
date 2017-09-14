---
layout: post
title: "A Gentle Introduction to Core Erlang: Part 1"
description: "An introduction to the Core Erlang language."
tags: [intro, core erlang, erlang, BEAM, compiler, program transformation]
comments: true
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

Back when I started my PhD (2.5 years ago), I was assigned to instrument Core
Erlang code. Since then, we have used Core Erlang in several other projects, but
people still ask me why we choose Core Erlang over Erlang.

# The Core Erlang language

In the beggining, working with Core Erlang proved to be quite a hard task due to
the lack of documentation, and I do not think it has improved much since then.
These entries are aimed towards providing a **basic understanding of the Core
Erlang language and its capabilities**.

In particular, this post introduces the Core Erlang language.

## Erlang made simpler

Core Erlang is a functional and concurrent programming language. In a nutshell,
it is a complete but simpler language than Erlang, since many Erlang constructs
are just syntactic sugar from a compiler perspective. In fact, Core Erlang is so
simple that we refer to it as _a subset of Erlang_ in many of our works.

Generally, Core Erlang is used as an intermediate language by the [Erlang
compiler](http://erlang.org/doc/man/compile.html).  In summary, **Erlang code is
translated to Core Erlang** before the generation of BEAM bytecode. The image
shown below illustrates this process. Here, blue elements are *human-readable*,
and red ones are not.

<figure>
  <img src="/images/intro-core-erlang-1/erlang-compile-diagram.png" alt="">
  <figcaption>Erlang code is translated to Core Erlang before the generation of BEAM bytecode.</figcaption>
</figure>

Actually, this diagram is a simplified version of the one in [Fonseca's thread
in the Elixir
forum](https://elixirforum.com/t/getting-each-stage-of-elixirs-compilation-all-the-way-to-the-beam-bytecode/1873/5)
which, in turn, could also be magnified in order to include other compilation
stages.

Therefore, developers are not expected to write Core Erlang code directly. This
code is usually generated instead, and any change you want to include can be
achieved by managing the resulting structure. Some of its features make it
**convenient for specific tasks**, but impractical for the rest of applications.

## From Erlang to Core Erlang

There are several ways to obtain the Core Erlang translation from an Erlang
source file.

### From the command line

If you only want to examine the Core Erlang code generated, this method is the
easiest one. Just type the following command in the command line:

```shell_session
erlc +to_core module.erl
```

The `+to_core` option will make the Erlang compiler to generate the
`module.core` file instead of `module.beam`.  If you open `module.core`, you
should be able to read the Core Erlang code translated from your module.

Alternatively, you can perform this translation from the Erlang shell. This
method is similar to the previous one, but I find that this way is more
convenient when you have to declare a large set of options:

```shell_session
c('module.erl', [to_core]).
```

The result will be the same as in the previous case.

### From an Erlang program

You can also get the associated Core Erlang code from within an Erlang program
using the `compile:file/2` function (you can check
[here](http://erlang.org/doc/man/compile.html#file-2) the available options and
possible outputs).

```erlang
case compile:file(File, [to_core]) of
    {ok, _, CoreForms} ->
      CoreForms;
    _Other ->
      io:fwrite("Error: Could not compile file.~n", []).
end.
```

In this case, the variable `CoreForms` stores the _forms_ that represent your
code, and you can **manage this data structure** in order to **perform analyses
or modifications in your program** (in the next part we will go into more detail
about this).

## An example

Let's consider the well-known factorial example, which can be written in Erlang
as:

```erlang
fact(0) -> 1;
fact(N) -> N * fact(N-1).
```

We can obtain the corresponding Core Erlang code by using any of the methods
previously mentioned.

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

Even though this is a basic example, you can already see why Core Erlang is much
simpler. Here are some things that happen when you translate Erlang into Core
Erlang:
 * **Pattern matching** is **moved** from anywhere **to case statements**.
 * **Guards** are **added to every clause** in a case statement.
 * **Function calls** (to external modules) and **function applications** (calls
   to the same module) have a **different syntax**.
 * **Function calls** (including built-in functions) are **fully qualified**.

Besides, a **catch-all clause** (i.e., a clause with a pattern `<X> when
'true'`) is added to **case statements** that do not include one.

These changes make Core Erlang more convenient for certain tasks. For instance,
**program transformation is easier to perform on Core Erlang programs**, since
there are fewer constructs and pattern matching always occurs in case
statements.

## Other resources

If you want to read more about the Core Erlang language, you can check these
resources:
 * [Richard Carlsson - An introduction to Core Erlang](http://citeseerx.ist.psu.edu/viewdoc/download;jsessionid=73993F4B68836CD1EF093D958523F16A?doi=10.1.1.111.6798&rep=rep1&type=pdf)
 * [Robert Virding - Implementing Languages on the BEAM](https://www.youtube.com/watch?v=qm0mbQbc9Kc)
 * [Carlsson et al. - CORE ERLANG 1.0.3 language specification](https://www.it.uu.se/research/group/hipe/cerl/doc/core_erlang-1.0.3.pdf)

Besides, I can not finish without mentioning [Kofi Gumbs - The Core of
Erlang](https://8thlight.com/blog/kofi-gumbs/2017/05/02/core-erlang.html). I
found his post when I started writing my own, and I used it as a reference
during the making of this post.

There, he goes into more detail on some points, but I wanted to keep this
introduction as simple as possible. In the next part, I intend to go a bit
further and explain how to manipulate Core Erlang programs in Erlang.

