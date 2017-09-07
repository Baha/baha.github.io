---
layout: post
title: "A Gentle Introduction to Core Erlang: Part 1"
description: "An introduction to the Core Erlang language."
tags: [intro, core erlang, erlang, program transformation]
comments: true
image:
  feature: abstract-5.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

# Core Erlang

```erlang
-module(factorial).
-export([fact/1]).

fact(0) -> 1;
fact(N) -> N * fact(N-1).
```

```
module 'factorial' ['fact'/1,
		    'module_info'/0,
		    'module_info'/1]
    attributes []
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
'module_info'/0 =
    fun () ->
	call 'erlang':'get_module_info'
	    ('factorial')
'module_info'/1 =
    fun (_@c0) ->
	call 'erlang':'get_module_info'
	    ('factorial', _@c0)
end
```

