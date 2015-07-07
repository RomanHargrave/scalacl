

# Prequisites #

  1. Make sure you've installed [Scala 2.9.x](http://www.scala-lang.org/)
  1. If you wish to use the [OpenCL-backed collections](ScalaCLCollections.md), ensure you have a working [OpenCL](http://www.khronos.org/opencl/) implementation :
    * [MacOS X 10.6 Snow Leopard](http://www.apple.com/macosx/technology/#opencl) comes with OpenCL pre-installed (won't work on earlier MacOS X versions)
    * [ATI Stream](http://www.amd.com/US/PRODUCTS/TECHNOLOGIES/STREAM-TECHNOLOGY/Pages/stream-technology.aspx) for ATI cards and/or any [SSE2](http://en.wikipedia.org/wiki/SSE2)-enabled CPUs on all flavours of Linux and Windows (even works on [Atom](http://en.wikipedia.org/wiki/Intel_Atom) CPUs !)
    * [NVIDIA drivers](http://www.nvidia.com/object/cuda_opencl_new.html) for NVIDIA cards
    * [Intel OpenCL SDK](http://software.intel.com/en-us/articles/intel-opencl-sdk/) for Intel CPUs on Windows (32/64 bits) or Linux (64 bits) ; beta version

<font color='red'><b>Warning :</b> ScalaCL is currently only tested on MacOS X, Windows 32/64 and Linux 64, <a href='http://groups.google.com/group/nativelibs4java'>feedback</a> on the other platforms would be much appreciated</font>

# The real stuff : OpenCL collections #

  1. Use the following `build.sbt` file (see [Usage](Usage.md) for other install methods) :
```
resolvers += Resolver.sonatypeRepo("snapshots")

resolvers += "NativeLibs4Java Repository" at "http://nativelibs4java.sourceforge.net/maven/"

libraryDependencies += "com.nativelibs4java" % "scalacl" % "0.2"

libraryDependencies += "com.nativelibs4java" % "javacl" % "1.0.0-RC2" // force latest JavaCL version

autoCompilerPlugins := true

addCompilerPlugin("com.nativelibs4java" % "scalacl-compiler-plugin" % "0.2")
```
  1. Create the following `Test.scala` file ([see supported syntax](CLConvertibleLanguageSubset.md)), or type the code in the Scala console (without the object and def definitions) :
```
import scalacl._
import scala.math._

object Test {
  def main(args: Array[String]): Unit = {
    implicit val context = Context.best // prefer CPUs ? use Context.best(CPU)
    val rng = (100 until 100000).cl // transform the Range into a CLIntRange
    // ops done asynchronously on the GPU (except synchronous sum) :
    val sum = rng.map(_ * 2).zipWithIndex.map(p => p._1 / (p._2 + 1)).sum
    println("sum = " + sum)
  }
}
```
  1. Compile and run (unless you're using the Scala console) :
```
scalac Test.scala
scala Test
```

Wanna know more ? See the [supported constructs](CLConvertibleLanguageSubset.md) and read the [FAQ](FAQ.md).

# Bonus : optimize loops on regular Scala collections #

If you don't want to use OpenCL-backed collections but are interested in making you regular Scala code faster, you can [have ScalaCL rewrite your array, list and (inline) range loops into much faster while loops](ScalaCLPlugin.md).

  1. Use the following `build.sbt` file (see [Usage](Usage.md) for other install methods) :
```
resolvers += Resolver.sonatypeRepo("snapshots")

resolvers += "NativeLibs4Java Repository" at "http://nativelibs4java.sourceforge.net/maven/"

autoCompilerPlugins := true

addCompilerPlugin("com.nativelibs4java" % "scalacl-compiler-plugin" % "0.2")
```
  1. Create the following `Test.scala` file :
```
object Test {
  def main(args: Array[String]): Unit = {
    val a = (100 until 100000).toArray
    val sum = a.map(_ * 2).zipWithIndex.map(p => p._1 / (p._2 + 1)).sum
    println("sum = " + sum)
  }
}
```
  1. Compile and run :
```
scalac Test.scala
scala Test
```

If you want to know what's happening (e.g. if there's an actual rewrite taking place), you can set the `SCALACL_VERBOSE` environment variable to `1` :
  * On Unix, Linux, MacOS X :
```
SCALACL_VERBOSE=1 scalac Test.scala
```
  * On Windows :
```
set SCALACL_VERBOSE=1
scalac Test.scala
```