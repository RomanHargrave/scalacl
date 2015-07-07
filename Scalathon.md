

I ([@ochafik](http://twitter.com/#!/ochafik)) was lucky enough to be invited for a presentation and many beers to [Scalathon](http://scalathon.org/), a community-driven event in Philadelphia (hosted at UPenn) organized with maestria by Brian Clapper, Yuvi Masori and an amazingly dedicated team.

Many people expressed interest in hacking with ScalaCL, so I've put up some basic information on this page to help them getting started.

# Documentation #

  * Olivier Chafik's [ScalaCL slides from Scalathon's speech](http://scalacl.googlecode.com/svn/wiki/ScalaCL%20Scalathon%20Philadelphia%202011.pdf)
  * Mirko Stocker's [Scala Refactoring](http://scala-refactoring.org/wp-content/uploads/scala-refactoring.pdf) master's thesis contains lots of easy to understand documentation about the compiler's AST in its Appendix D.
  * anything missing ? come and ask !

# Cheat sheet #

## Scalac options ##

```
scalac -Xprint:typer test.scala
```


```
scalac -Xprint:typer -Yshow-trees test.scala
```

```
scalac -Ybrowse:typer test.scala
```

You can replace `typer` by any other phase, such as `scalacl-loopstransform`.

## Run the compiler with the plugin ##

With sbt 0.10.1 :
```
sbt "run test.scala"
```
```
sbt "run test.scala -cp mylib.jar ..."
```

With maven (you'll have to edit the `scalacl` script file to add JAR dependencies, since this will only take one file argument) :
```
./scalacl test.scala
```

## Workflow ##

  * Use sbt to ~compile or ~run your code as you write it
  * Use maven to run tests (or help fix scalacl's tests with sbt 0.10.1 :-))

# What can you hack ? #

  * moving code generation from runtime library to compiler plugin : look for `println("sourceData = " + sourceData)` in `ScalaCLFunctionsTransformComponent.scala`, and for `CLFunction`'s constructor : constructor needs to be modified to take the actual OpenCL code, rather than the normalized code's intermediate form
  * **MOSTLY DONE** side-effects detection : see code in `CodeAnalysis.scala` ; this is needed for streams rewrites
  * symmetry detection : copy paste code from side-effects detection and get going :-)
  * sbt migration `sbt test` fails, `mvn test` succeeds (except for side effects tests)
  * write your own plugin component : look at `MyComponent.scala`, then think of registering your component in `ScalaCLPlugin.scala`
  * auto-vectorization :
    1. detect `CLArray.apply` "seeds" calls
    1. traverse up from these calls until you reach an `IntRange` + `Foreach` loop (see `MiscMatchers.scala`)
    1. see if the foreach is convertible to OpenCL (checking captured variables... see convertFunctionToCLFunction
    1. output some parallel code : maybe by building a `CLRange`...
  * add runtime + compilation support for images :
```
trait CLImage2D[PixelType] {
  lazy val img: com.nativelibs4java.opencl.CLImage2D
  def apply(x: Int, y: Int): PixelType
  def update(x: Int, y: Int, value: PixelType)
}
```
> Also see [CLImage2D](http://nativelibs4java.sourceforge.net/javacl/api/current/com/nativelibs4java/opencl/CLImage2D.html) in [JavaCL](http://code.google.com/p/javacl/) (have to be careful reading / writing...)
    * detect image reads/writes in code to say whether it's gonna be used as OpenCL input or output
    * update `OpenCLConverter` to detect calls to `CLImage2D[T].apply` / `update` and replace them by [read\_imagei](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/read_imagei2d.html) or [read\_imagef](http://www.khronos.org/registry/cl/sdk/1.0/docs/man/xhtml/read_imagef2d.html) / write\_image... See the `case Apply(Select(target, applyName()), List(singleArg)) =>` line
    * implement the apply and update methods with calls to [long, long, long, long, long, org.bridj.Pointer, boolean, com.nativelibs4java.opencl.CLEvent...) CLImage2D.read](http://nativelibs4java.sourceforge.net/javacl/api/current/com/nativelibs4java/opencl/CLImage2D.html#read(com.nativelibs4java.opencl.CLQueue,)... and dealing with the different image formats :-S
- swallow functions defined in same compilation unit
- simple lamda lift : forward captured symbols
- fix side-effects detection
- speedup tests / fix sbt tests
- find a way to debug-break into the plugin in any IDE !

  * other ideas ? (look at [the slides](http://scalacl.googlecode.com/svn/wiki/ScalaCL%20Scalathon%20Philadelphia%202011.pdf) :-))