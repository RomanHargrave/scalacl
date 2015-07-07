

# What is ScalaCL ? #

Two things :
  * a [Scala compiler plugin](ScalaCLPlugin.md) that optimizes general loops on arrays, lists and (inline) int ranges. For instance, the following code will run faster with [ScalaCLPlugin](ScalaCLPlugin.md) :
```
for (i <- 0 to 10000)
  println(i)
```
> The loop **optimizations don't rely on any library or hardware** : it's rewriting of pure Scala into faster pure Scala.
  * a [library of OpenCL-backed collections](ScalaCLCollections.md) that can run some of their operations directly on the GPU (mostly filter and map) :
```
import scalacl._
import scala.math._
implicit val context = new ScalaCLContext
val r = (0 to 1000000).cl // this is a CLRange
val a = r.toCLArray // this is a CLArray[Int]
val m = a.map(v => cos(v).toFloat) // run asynchronously on the GPU via OpenCL
println(m)
```
> The ScalaCL collections make use of the compiler plugin to translate Scala functions into OpenCL code.

Please read [Usage](Usage.md) to know how to install/use ScalaCL in your projects.

# A Scala expression fails to be converted by the plugin... Which are supported ? #

For Scala to OpenCL code conversion, please see [this list of supported constructs](CLConvertibleLanguageSubset.md).

# Does the ScalaCL Compiler Plugin add a dependency to any OpenCL library / hardware to my Scala code ? #

**No.**

If you're just using the [compiler plugin](ScalaCLPlugin.md) on regular Scala code to optimize your loops on arrays, ranges and lists, then the plugin won't snoop in any dependency into your code : it will just rewrite functional style "loop-ish" operations into fast while loops.

If, however, you're explicitely using the [OpenCL collections](ScalaCLCollection.md) provided by ScalaCL, then you'll need an OpenCL implementation at runtime.

# Why are ScalaCL Collections asynchronous ? What does it mean ? #

To the end user, [ScalaCLCollections](ScalaCLCollections.md) appear just as any other synchronous Scala collection.

However, when you run `CLArray(1, 2, 3).map(_ + 1).map(_ * 2)`, the two map operations are not run synchronously. This means they might not have even been started yet !

The returned collection knows about which asynchronous operations are pending, and it will wait for them to complete if, say, you want to run `.toArray` on it.

# How do I know for sure that things worked well ? #

If you want to know what's happening (e.g. if there's an actual rewrite taking place during compilation or if the OpenCL collections actually schedule OpenCL kernels for execution), you can set the `SCALACL_VERBOSE` environment variable to `1` :
  * On Unix, Linux, MacOS X (bash) :
```
export SCALACL_VERBOSE=1
```
  * On Windows :
```
set SCALACL_VERBOSE=1
```

Also, you can ask `scalac` to show you the source after the [ScalaCLPlugin](ScalaCLPlugin.md) transforms :
  * For general Scala collections transforms :
```
scalac Test.scala -Xprint:scalacl-loopstransform
```
  * For OpenCL collections transforms :
```
scalac Test.scala -Xprint:scalacl-functionstransform
```

# I don't get it : JavaCL is LGPL-licensed and ScalaCL is BSD-licensed ? #

Short answer : **don't worry, ScalaCL is BSD-licensed**.

The current main distribution of [JavaCL](http://code.google.com/p/javacl/) relies on [JNA](https://jna.dev.java.net/) for its low-level bindings, and JNA is licensed under the LGPL license. The JNA version of JavaCL is hence released under the LGPL license for simplicity.

However, ScalaCL uses an alternate port of JavaCL that uses [BridJ](http://code.google.com/p/bridj/) as its underlying low-level binding technology instead of JNA, and BridJ is licensed under a BSD license, so I chose to release this JavaCL port and ScalaCL under BSD as well (being the author of BridJ, JNAerator, JavaCL and ScalaCL and relying on open-licensed projects such as [dyncall](http://www.dyncall.org) leaves me some licensing flexibility :-)).

Please see the [Credits and License](http://code.google.com/p/bridj/wiki/CreditsAndLicense) page for more details.

# Where can I get some help with ScalaCL ? #

  * On the [NativeLibs4Java user group](http://groups.google.com/group/nativelibs4java).
> [NativeLibs4Java](http://code.google.com/p/nativelibs4java/) is the umbrella project of [JavaCL](http://code.google.com/p/javacl/), ScalaCL, [BridJ](http://code.google.com/p/bridj/) and [JNAerator](http://code.google.com/p/jnaerator/).
  * On [Twitter : @ochafik](http://twitter.com/ochafik)

# I get a `java.lang.ClassNotFoundException: scala.Serializable` #

You're probably not using the same version of Scala that ScalaCL was built for.

Please have a look at ScalaCL's [prequisites](Usage.md).

# What about List rewrites ? #

Rewriting loop operations on Lists into while loops does not always result in a speedup.

In fact, a simple foreach on a List won't be faster once rewritten into a while loop in most cases. That's why List rewrites were disabled in ScalaCL 0.2 by default.

However, ScalaCL 0.3-SNAPSHOT supports streams rewrites, so List(1, 2, 3).map(...).foreach(...).max will be rewritten in a single while loop (if there are no side-effects in the closures), and in fact the data structure will even be switched to Array(1, 2, 3), which is faster to create and iterate upon.

You can have a look at the code that decides whether a stream rewrite is worth the pain (look at the List case : it takes 2 closures at least in the stream) :
https://github.com/ochafik/nativelibs4java/blob/master/libraries/OpenCL/ScalaCLPlugin/src/main/scala/scalacl/plugin/StreamTransformComponent.scala#L297