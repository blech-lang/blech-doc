---
title: "Singletons and globals"
linkTitle: "Singleton globals"
draft: true
weight: 30
description: >
  Evolution proposals for the Blech language.
---

{{% pageinfo %}}
The singleton globals proposal is work in progress.
{{% /pageinfo %}}
.
## Singletons 

A `singleton` in Blech is a `function` or an `activity` that cannot be used concurrently in order to prevent a concurrent write-write or read-write conflict.

Singletons originate from functions or activities that are allowed to change a global in the environment.   
If a subprogram - a `function` or an `activity` - declares an `extern var` the compiler overestimates that such a function or activity accesses a global in the environment even if the `extern var` is not changed in the code. The subprogram becomes a `singleton function` or a `singleton activity`.
Note that even if the function or activity only reads the global, there is a risk of read-write data races if the global is not synchronised with the activity, such as memory-mapped external sensors. 

The use of a `singleton` creates a dependent singleton. If a subprogram `caller` calls or runs a singleton subprogram `callee`,  `caller` becomes a dependent singleton function `singleton [callee] function caller` or activity `singleton [callee] activity caller`.
Subprogram `caller` cannot be used concurrently with itself, with subprogram `callee` and with any other singleton that depends on `callee`.

If a Blech program uses an `extern function` that changes a global in the enviroment, it should be declared as `extern singleton function` in order to prevent a concurrent write-write conflict to the environment.

If a Blech program uses an `extern function f` and an `extern function g` that both change the same global in the environment the programmer can prevent a concurrent write-write conflict by introducing an opaque `singleton name` and declaring the extern functions as dependent singletons `extern singleton [name] function f` and `extern singleton [name] function g`. Now `f` and `g` cannot accidentally be called concurrently because both depend on the same opaque singleton.

Singletons are infectious for their callers and therefore have to be used with great care. Subprograms are only reusable in different concurrent situations if they are not singletons.


## Globals and singletons - two sides of the same coin.

Global variables have a similar effect as singletons. Two subprograms that write to the same global variable cannot be used concurrently. Therefore Blech does not allow global variables and only has `extern var` declarations local to subprograms.

### Why globals might be useful

Today, every information that the environment needs from the Blech program, has to be made available via output parameters of the Blech programs `@[EntryPoint] activity`. The output parameters publish the information that might be consumed by the environment.

If the Blech program "knows" what the environment needs, it can also write to an `extern var` or call an `extern singleton function` that writes to the environment.

Nevertheless there is no interface that allows the environment to call into the Blech program in order to get an information offered by a more flexible API.




