**Project is being rewritten from scratch using Scala 2.10 macros: [ScalaCL on GitHub](https://github.com/ochafik/ScalaCL).**

<font color='red'><b>Warning :</b> ScalaCL might still have (unknown) bugs**, it's currently far from being finished (especially on the OpenCL side)</font>.**

However, there are some happy production users out there of the general loops rewrite plugin feature, so it's not all sadness and sorrow.

_Feed this to your graphic card : `a.map(v => (v, cos(v)))`, and [much more](CLConvertibleLanguageSubset.md)_

[FAQ](FAQ.md) | [Getting Started in 3 minutes-chrono](GettingStarted.md) | [Get / Install](Usage.md) | [Architecture](Architecture.md) | [Scalathon](Scalathon.md)

# About ScalaCL #

ScalaCL lets programmers **run Scala code on GPUs** in a very natural way (using [JavaCL bindings](http://code.google.com/p/javacl/) to the [OpenCL API](http://www.khronos.org/opencl/)).

It also **optimizes general Scala loops (on arrays, lists and inline ranges)** often by a big margin so you'll want to use it even if you don't care about OpenCL.

**ScalaCL is not production-ready !** Please help us by [testing](Usage.md) it, giving us [feedback](http://groups.google.com/group/nativelibs4java) and filing [issues](Issues.md).

ScalaCL is [architectured](Architecture.md) around two components :
  * [ScalaCL Collections](ScalaCLCollections.md) : OpenCL-backed collections that look and behave like standard Scala collections (work in progress).
  * [ScalaCL Compiler Plugin](ScalaCLPlugin.md) : optimizes Scala programs at compile-time, transforming regular Scala loops into faster code and transforming Scala functions given to [ScalaCL Collections](ScalaCLCollections.md) into OpenCL kernels (work in progress, [see supported syntax](CLConvertibleLanguageSubset.md))

ScalaCL is part of the [NativeLibs4Java ecosystem](http://code.google.com/p/nativelibs4java/), which comprises a native bindings generator ([JNAerator](http://code.google.com/p/jnaerator/)), a native interoperability runtime ([BridJ](http://code.google.com/p/bridj/)), OpenCL bindings ([JavaCL](http://code.google.com/p/javacl/)) and more...

# Learning Scala #

You're interested in using ScalaCL but you don't know Scala ?
Besides the excellent [Learning Scala](http://www.scala-lang.org/node/1305) section of the Scala website, I'd suggest the following two sources :
  * [Scala for Java Refugees](http://www.codecommit.com/blog/scala/roundup-scala-for-java-refugees)
  * [Simply Scala](http://www.simplyscala.com/)

Happy learning !