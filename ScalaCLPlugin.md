

# What is it ? #

The ScalaCL Compiler Plugin makes your code run faster by transforming it at compile-time.

It features general optimizations (not tied to [OpenCL](http://www.khronos.org/opencl/) whatsoever) and [OpenCL](http://www.khronos.org/opencl/)-specific optimizations.

## OpenCL-specific optimizations ##

The plugin transforms inline Scala functions in `CLCollection.map` and `CLCollection.filter` into [OpenCL](http://www.khronos.org/opencl/) kernels, effectively running these functions on the GPU via [OpenCL](http://www.khronos.org/opencl/) (more precisely, it converts these functions into _augmented functions_ that are proxies to the original functions and contain an OpenCL conversion of the functions' Scala code).

The programmer must explicitely use [ScalaCL Collections](ScalaCLCollections.md) to use this feature, which is **not yet stable** (see the [Usage](Usage.md) and [GettingStarted](GettingStarted.md) pages for how to install and/or use the collections).

## General optimizations ##

The ScalaCL Compiler Plugin optimizes simple generalistic Scala loop-like calls by transforming them into equivalent while loops, which are much faster.

In other terms : **it makes your generalistic Scala code faster for free**.

You don't need [ScalaCL Collections](ScalaCLCollections.md) to benefit from these optimizations and the plugin does not add any dependency to your optimized code (it's just free performance gain, for real :-)).

Here are the methods currently optimized by the plugin :

| **Method** | **`Array[T]`** | **`List[T]`** | **`Option[T]`** | **inline range**<br />`x to/until y [by z]` (with filters) |
|:-----------|:---------------|:--------------|:----------------|:-----------------------------------------------------------|
| `foreach`  | `*`            |               |                 | `*`                                                        |
| [tabulate](http://www.scala-lang.org/api/current/scala/Array$.html) | `*`            |               |                 |                                                            |
| `map`      | `*`            |               |                 | `*`                                                        |
| `reverseMap` |                |               |                 |                                                            |
| `sum`      | `*`            |               |                 | `*`                                                        |
| `min`      | `*`            |               |                 |                                                            |
| `max`      | `*`            |               |                 |                                                            |
| `mkString` |                |               |                 |                                                            |
| `reduceLeft` | `*`            |               |                 |                                                            |
| `reduceRight` | `*`            |               |                 |                                                            |
| `foldLeft` (equiv. to `/:`) | `*`            |               |                 | `*`                                                        |
| `foldRight` (equiv. to `:/`) | `*`            |               |                 |                                                            |
| `scanLeft` | `*`            |               |                 | `*`                                                        |
| `scanRight` | `*`            |               |                 |                                                            |
| `forall`   | `*`            |               |                 | `*`                                                        |
| `exists`   | `*`            |               |                 | `*`                                                        |
| `count`    | `*`            |               |                 | `*`                                                        |
| `prefixLength` |                |               |                 |                                                            |
| `partition` |                |               |                 |                                                            |
| `span`     |                |               |                 |                                                            |
| `find`     |                |               |                 |                                                            |
| `flatMap`  |                |               |                 |                                                            |
| `filter`   | `*`            |               |                 | `*`                                                        |
| `filterNot` | `*`            |               |                 | `*`                                                        |
| `prefixLength` |                |               |                 |                                                            |
| `indexWhere` |                |               |                 |                                                            |
| `lastIndexWhere` |                |               |                 |                                                            |
| `takeWhile` | `*`            |               |                 | ...                                                        |
| `dropWhile` | `*`            |               |                 | ...                                                        |

Examples of supported generalist rewrites can be found in the automatic tests :
https://github.com/ochafik/nativelibs4java/tree/master/libraries/OpenCL/ScalaCLPlugin/src/test/scala/scalacl/

You can also read this thread of the scala-user mailing-list where I announced the plugin :

[ScalaCL Compiler Plugin to optimize loops on arrays and int ranges (out for testing and feedback)](http://scala-programming-language.1934581.n4.nabble.com/ScalaCL-Compiler-Plugin-to-optimize-loops-on-arrays-and-int-ranges-out-for-testing-and-feedback-td2953642.html)

# Using the compiler plugin #

Please see the [Usage](Usage.md) page.

# Benchmarking #

If you want to compare the performance of Scala programs with and without the ScalaCL Compiler plugin, you can :
  * compile twice the same program : once with the `DISABLE_SCALACL_PLUGIN=1` environment variable set, and once without.
  * put twice the same code in two separate classes and files (`TestWithout.scala` and `TestWith.scala`), then set the `SCALACL_SKIP=TestWithout` environment variable when compiling : the `TestWithout.scala` file won't benefit from any optimization from the compiler plugin.

Also, please think of always using the `-optimise` switch of [scalac](http://www.scala-lang.org/docu/files/tools/scalac.html). You'll notice that it does not reduce the advantage of using the ScalaCL compiler plugin, on the contrary : in many cases the plugin makes it easier for the compiler to optimize the program.

Here's an example of code that gets a x30 boost with ScalaCL's Compiler Plugin (depending on your JVM settings and on `n`) :
```
// Naive dense matrix multiplication
val n = 100
val a = Array.tabulate(n, n)((i, j) => (i + j) * 1.0)
val b = Array.tabulate(n, n)((i, j) => (i + j) * 1.0)
val o = Array.tabulate(n, n)((i, j) => (0 until n).map(k => a(i)(k) * b(k)(j)).sum)
```
See [some examples of programs](https://github.com/ochafik/nativelibs4java/tree/master/libraries/OpenCL/ScalaCLTest/) that benefit from being optimized by the ScalaCL Compiler Plugin.

# Project #

You can [follow news about the ScalaCL plugin on Twitter (@ochafik)](http://twitter.com/#!/ochafik).

There's also the [TODO](TODO.md) page if you want to see the next priorities.

## Compiling Scala with ScalaCL ##

ScalaCL is now capable of optimizing Scala + Scalac (versions 2.8.x). Please note however that this does not seem to speed up Scala(c) at all, as little optimizations are done (about 200 in the whole Scala source tree). Next versions (including the latest development version) perform far more optimizations, but please be patient until they're stable enough :-)
```
svn co http://lampsvn.epfl.ch/svn-repos/scala/scala/branches/2.8.x scala-2.8.x
cd scala-2.8.x
mkdir -p misc/scala-devel/plugins/
cd misc/scala-devel/plugins/
wget http://nativelibs4java.sourceforge.net/maven/com/nativelibs4java/scalacl-compiler-plugin/0.1/scalacl-compiler-plugin-0.1.jar
# Or the latest development version :
# wget http://nativelibs4java.sourceforge.net/maven/com/nativelibs4java/scalacl-compiler-plugin/1.0-SNAPSHOT/scalacl-compiler-plugin-1.0-SNAPSHOT.jar
cd ../../..
ant clean build
```
Then you can run tests :
```
ANT_OPTS="-Xmx2g -Xms1g -XX:MaxPermSize=500m" ant test
```
And to recompile without any optimization :
```
DISABLE_SCALACL_PLUGIN=1 ant clean build
```

## Where is the source code ? ##

[ScalaCL is licensed under a 3-clause BSD-style license](https://github.com/ochafik/nativelibs4java/tree/master/libraries/OpenCL/ScalaCLPlugin/LICENSE), so it can be integrated in pretty much all living software (including proprietary and (L)GPL).

  * [ScalaCL Compiler Plugin](https://github.com/ochafik/nativelibs4java/tree/master/libraries/OpenCL/ScalaCLPlugin/)
  * [ScalaCL Collections](https://github.com/ochafik/nativelibs4java/tree/master/libraries/OpenCL/ScalaCL2/)

To start hacking on the compiler plugin :
```
svn co http://nativelibs4java.googlecode.com/svn/trunk/libraries/OpenCL/ScalaCLPlugin
cd ScalaCLPlugin
mvn package
```
Also see [this page](CompileScalaCL.md) for how to compile [ScalaCL Collections](ScalaCLCollections.md) as well.

## Automated Tests ##

Most tests are run with :
```
mvn test
```

However, the PerformanceTest is much slower and is disabled by default (it does a microbenchmark for nearly all the optimized cases, with varying collection sizes, cold vs. warm tests, optimized vs. non-optimized comparison...) :
```
SCALACL_TEST_PERF=1 mvn test -Dtest=PerformanceTest
```