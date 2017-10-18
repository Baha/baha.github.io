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

In general, it is more convenient to perform source-to-source transformation in Core Erlang than carrying out this task in Erlang if our goal is to generate BEAM bytecode. This is due, in part, to the simplicity of the Core Erlang language.

But, it can become an arduous task at certain times, and the scarce and outdated documentation available does not make it better. Moreover, it is quite difficult to find examples of how to write code for this. The first time I worked on Core Erlang instrumentation, I spent most of the time browsing code on GitHub and developing code by trial and error.

## Abstract Syntax Trees

As it is common in programming languages, a Core Erlang program can be represented with an *Abstract Syntax Tree* (AST for short). The nodes in this tree represent the constructs that occur in the source code. In this context, the terms *AST* and *forms* are interchangeable.

<figure>
  <img src="/images/intro-core-erlang-1/core-erlang-ast.png" alt="">
  <figcaption>The Abstract Syntax Tree for the factorial program in Core Erlang.</figcaption>
</figure>

For example, the AST that represents the factorial module is a module declaration (root node) that defines a single function (the child subtree) that is let expression with three subtrees (one for its variables, one for its argument and one for the body).

In general, the number of subtrees for a particular node might be variable. Hence, the arity in ASTs is not fixed for some constructs. For example, a module declaration consists of several functions definitions (i.e., a module node has a child node for each function definition), but a let expression will always have three subtrees.

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

The type of `Node` can be obtained with the `cerl:type/1` function. Another option is to match with the record type of `Node` directly (`#c_apply{...}`, `#c_call{...}`, ...), but I prefer the former option because I think it is more readable in general.

## Metaprogramming in Erlang

In this case, the metaprogramming process can be divided into three steps:
 * Translate Erlang source code to Core Erlang forms.
 * Manipulate the Core Erlang forms.
 * Generate Core Erlang code from forms.

These three steps can be summarized as *read*, *modify* and *write* if you regard code and forms two representations of the same thing: A program. In addition, I will also describe a last step: Compiling the Core Erlang code (i.e., the code that we have written) to generate the BEAM file.

### From Erlang code to Core Erlang forms

In the last part, we saw how to get the *forms* associated to the Core Erlang code using the 
`compile:file/2` function.

```erlang
case compile:file(File, [to_core, binary, no_copt]) of
    {ok, _, CoreForms} ->
      CoreForms;
    _ ->
      io:fwrite("Error: Could not compile file.~n").
end.
```

We already know that the `to_core` option in the `compile:file/2` call is used to obtain the Core Erlang forms, but there are some other options that can be useful:
 * `binary`: By default, `compile:file` generates a file (the result of the compilation). With this option, we avoid its generation.
 * `no_copt`: Disables compiler optimizations.

If we aim at generating a modified version of our program and we are not interested in the unchanged version, then we should include the `binary` option.

Occasionally, the translated forms do not have a clear correspondence with the original Erlang code (because of compiler optimizations). In these cases, you can use `no_copt` to obtain forms that are closer to original code.

### Manipulation of Core Erlang forms

Once you have read the forms, it is possible to perform the transformation node by node, starting by the module node, and propagating it to the subnodes. This way, you get full control over the transformations you perform on each node, and whether you propagate or not these transformation to its subtrees.

However, if your transformation is going to focus on a particular type of node, I would recommend using the `cerl_trees:map/2` function instead. `cerl_trees:map/2` receives a tree and a function, and it traverses the tree applying this function to each node.

Suppose we have been asked to replace all the appearances of the concatenation operations (here, a call to the `++/2` function) for the (i.e., convert `[X] ++ Xs` to `[X|Xs]`).

```erlang
replace_concat(Node) ->
  case cerl:type(Node) of
    call ->
      ConcCallName = cerl:concrete(cerl:call_name(Node)),
      case ConcCallName of
        '++' ->
          [FstArg, SndArg] = cerl:call_args(Node),
          FlatFstArg = cerl:cons_hd(FstArg),
          cerl:c_cons(FlatFstArg,
                      SndArg);
        _ -> Node
      end;
    _ -> Node
  end.
```

This code might seem a bit confusing at first sight, but...

Of course, we could add further code in order to check that the `FstArg` is a list with a single element.

### From Core Erlang forms to code

Then, we must write the transformed Core Erlang forms into a file. This is as simple as writing a file that contains the Core Erlang code. We can help ourselves by using the `cerl_prettypr:format/1` function.
```erlang
file:write_file("transformed.core",
                cerl_prettypr:format(InstCoreForms)).
```

### From Core Erlang code to BEAM bytecode

Compiling a Core Erlang file works the same way as compiling an Erlang file, with the difference that you must specify `+from_core` as an option when you run the compiler:

```shell_session
erlc +from_core module.core
```

The same thing applies to compiling from the Erlang shell:

```shell_session
c('module.core', [from_core]).
```

Lastly, you can also use the `compile:file/2`

## Other things to consider

### Annotations

Apart from the data type and the subtrees in a node, there is another field dedicate to annotations. By default, annotations contain the name of the file and the corresponding line in the Erlang source code.

Annotations can be quite useful is some cases. For instance, if we have to perform an analysis in the Core Erlang code prior to a second pass algorithm, we can use the annotations to store any data that could be useful during this second pass.

### Libraries

## Other resources

As I mentioned at the beginning, there are not many resources to check for Core Erlang metaprogramming.
 * Erlang docs (cerl and others)
* [Salvador Tamarit - ]()
* [Salvador Tamarit - Metaprogramming for Erlang Abstract Format & Core]()

 * merl
 * smerl
