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

In general, it is more convenient to perform source-to-source transformation in Core Erlang than carrying out this task in Erlang when our goal is to generate BEAM bytecode. This is due, in part, to the simplicity of the Core Erlang language.

But, it can become an arduous task at certain times, and the scarce and outdated documentation available does not make it better. Moreover, it is quite difficult to find examples of how to write code for this. Actually, when I started to write Erlang code for instrumentation, I started by browsing code on Github, and the major part developing was trial & error :wink:

## Abstract Syntax Trees

As it is usual in programming languages, Core Erlang programs are represented with an Abstract Syntax Tree (AST for short). 

<figure>
  <img src="/images/intro-core-erlang-1/core-erlang-ast.png" alt="">
  <figcaption>The Abstract Syntax Tree for the factorial program in Core Erlang.</figcaption>
</figure>

For example, the AST that represents the factorial module is a module declaration (root node) that defines a single function (the child node) that first executes a let statement with the argument.

Arity.

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

The metaprogramming process can be divided into 3 parts:
 * Read the forms
 * Manipulate the forms
 * Write the code

Optionally...

### Read

In the last part, we already saw how to get the *forms* associated to the Core Erlang code (i.e, the AST) 

```erlang
case compile:file(File, [to_core, binary]) of
    {ok, _, CoreForms} ->
      CoreForms;
    _ ->
      io:fwrite("Error: Could not compile file.~n").
end.
```

We already know that the `to_core` option in the `compile:file/2` call is used to obtain the Core Erlang forms, but there are some other options that can be useful:
 * `binary`: By default, `compile:file` generates a file that contains the result. This option tells the compiler to not generate this file.
 * `no_copt`: Sometimes, the compiler tries to optimize your code. This option disenables compiler optimizations.

 Obviously, if we are going to generate a modified program...

### Manipulate

Once you have read the forms, it is possible to perform the transformation node by node, starting by the module node, and propagating it to the subnodes.

However, if your transformation is going to focus on a particular type of node, I would recommend using the `cerl_trees:map/2` function.

### Write

```erlang
  file:write_file("transformed.core",
                  cerl_prettypr:format((cerl_trees:map(Stripper, InstCoreForms)))).
```

### Compile & Execute

```shell_session
erlc +from_core module.core
```

Or, if you prefer it, you can compile it from the Erlang shell.

```shell_session
c('module.core', [from_core]).
```

## Other things to consider

### Annotations

Apart from the data type and the subtrees in a node, there is another field dedicate to annotations. By default, annotations contain the name of the file and the corresponding line in the Erlang source code.

Annotations can be quite useful is some cases. For instance, if we have to perform an analysis in the Core Erlang code prior to a second pass algorithm, we can use the annotations to store any data that could be useful during this second pass.

### Libraries

## Other resources

The documentation about... 
 * Erlang docs (cerl and others)

[Salvador Tamarit - Metaprogramming for Erlang Abstract Format & Core]()

 * merl
 * smerl
