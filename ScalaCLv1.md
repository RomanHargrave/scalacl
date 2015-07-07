<font color='red'><b>Important:</b> ScalaCLv1 is no longer maintained, all development efforts are  put on ScalaCLv2</font>

ScalaCLv1 was a simple DSL for (limited) parallel expressions execution on GPUs (proof-of-concept quality).

It was introduced in this blog post :

[ScalaCL: Reap OpenCL’s benefits without learning its syntax (Scala DSL for transparently parallel computations)](http://ochafik.free.fr/blog/?p=207)

Here's a simple example :
```
class VectorAdd(i: Dim) extends Program(i) {
	val a = FloatsVar
	val b = FloatsVar
	var result = FloatsVar
	content = result := a + b
}
```
[See the complete example...](http://nativelibs4java.googlecode.com/svn/trunk/libraries/OpenCL/ScalaCL/src/main/scala/scalacl/ScalaCLTest.scala)