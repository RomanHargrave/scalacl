

You can use the ScalaCLPlugin with **any of the following options** : sbaz, sbt, Maven, eclipse...

# Prequisites #

  * OpenCL : If you want to use [ScalaCL Collections](ScalaCLCollections.md), you need to [install an OpenCL implementation](http://code.google.com/p/javacl/wiki/Usage#OpenCL_implementation).
> You don't need any OpenCL implementation for the general loops rewrite stuff !
  * Scala : ScalaCL was built with and for Scala 2.9.x (won't work with Scala 2.8.x unless you [build ScalaCL from sources](Build.md) or [donate to the project](http://sourceforge.net/donate/index.php?group_id=266856) so some legacy target is deployed for you ;-))

# With scalac (using sbaz) : preferred method #

## Install ##

If you're using [scalac](http://www.scala-lang.org/docu/files/tools/scalac.html) in command line, the easiest way to test ScalaCL's compiler plugin is to use the following [sbaz](http://www.scala-lang.org/node/93) commands :
```
sbaz update
sbaz install scalacl
```
Then just use `scalac` as you're used to (the plugin will be enabled by default, see below for how to disable it). You don't need to specify the classpath of the [ScalaCLCollections](ScalaCLCollections.md), they should be readily available and automatically imported in `$SCALA_HOME/lib`.

## Uninstall ##

Should you wish to uninstall ScalaCL collections and compiler plugin, run :
```
sbaz remove scalacl
```

## Update ##

To update ScalaCL collections and compiler plugin :
```
sbaz update
sbaz remove scalacl
sbaz compact
sbaz install scalacl
```
Or if you don't care upgrading all your sbaz-managed packages :
```
sbaz update && sbaz upgrade
```

## Snapshots (development versions) ##

In addition to release and beta versions, some snapshots are regularly shared on sbaz. To list the available versions :
```
sbaz update
sbaz available -a
```
Look for the line that looks like :
```
scalacl (0.2.final, 0.2.Beta14, 0.2.Beta13, 0.2.Beta2, 0.2-SNAPSHOT_20110112_1048, ...)
```
You can then install a particular version with :
```
sbaz remove scalacl
sbaz install scalacl/0.2.final
```

# With scalac (manually) #

  * Download the latest version of ScalaCL Plugin :
> [scalacl-compiler-plugin-0.3-SNAPSHOT.jar](http://oss.sonatype.org/content/groups/public/com/nativelibs4java/scalacl-compiler-plugin/0.3-SNAPSHOT/) (make sure to download the latest snapshot)
  * Rename the timestamped snapshot file exactly to `scalacl-compiler-plugin-0.3-SNAPSHOT.jar` (i.e. replace the timestamp by `SNAPSHOT`), otherwise the compiler plugin mechanism won't work :-S
  * Make sure you've uninstalled any version of ScalaCL Plugin previously installed with sbaz :
```
sbaz remove scalacl
```
  * Use it like this (please do not rename the JAR or it won't work)
```
scalac -Xplugin:scalacl-compiler-plugin-0.3-SNAPSHOT.jar ...
```

# With sbt #

## sbt 0.10.x ##

To use ScalaCL's collections and/or compiler plugin with [sbt](http://code.google.com/p/simple-build-tool/) 0.10.x, please make your `build.sbt` file look like this :
```
resolvers += "Sonatype OSS Snapshots Repository" at "http://oss.sonatype.org/content/groups/public/"

resolvers += "NativeLibs4Java Repository" at "http://nativelibs4java.sourceforge.net/maven/"

libraryDependencies += "com.nativelibs4java" % "javacl" % "1.0.0-RC2" // force latest JavaCL version

libraryDependencies += "com.nativelibs4java" % "scalacl" % "0.2"

autoCompilerPlugins := true

addCompilerPlugin("com.nativelibs4java" % "scalacl-compiler-plugin" % "0.2")
```

## sbt 0.7.x ##

For sbt 0.7.x versions, here's what your project file (say, `project/build/ScalaCLTest.scala`) should look like :
```
import sbt._
class ScalaCLTest(info: ProjectInfo) extends DefaultProject(info) with AutoCompilerPlugins
{
  val sonatypeSnapshotsRepo = "Sonatype OSS Snapshots Repository" at "http://oss.sonatype.org/content/groups/public/"
  val nativelibs4javaRepo = "NativeLibs4Java Repository" at "http://nativelibs4java.sourceforge.net/maven/"
  // Current preferred beta version :
  val scalaclPlugin = compilerPlugin("com.nativelibs4java" % "scalacl-compiler-plugin" % "0.2")
  val scalacl = "com.nativelibs4java" % "scalacl" % "0.2"
  // Current development version : 
  //val scalaclPlugin = compilerPlugin("com.nativelibs4java" % "scalacl-compiler-plugin" % "0.3-SNAPSHOT")
  //val scalacl = "com.nativelibs4java" % "scalacl" % "0.3-SNAPSHOT"
}
```

Of course, do not forget to update to grab the package(s) :
```
sbt update
```

# With Maven #

The ScalaCL plugin can be used with [Maven](http://maven.apache.org/), simply edit your `pom.xml` as follows :
```
...
  <repositories>
    ...
    <repository>
      <id>sonatype</id>
      <name>Sonatype OSS Snapshots Repository</name>
      <url>http://oss.sonatype.org/content/groups/public</url>
    </repository>
    <repository>
      <id>nativelibs4java</id>
      <name>nativelibs4java Maven2 Repository</name>
      <url>http://nativelibs4java.sourceforge.net/maven</url>
    </repository>
  </repositories>

  <dependencies>
    <!-- Not needed if you do not care about OpenCL collections and just need general optimizations : -->
    <dependency>
      <groupId>com.nativelibs4java</groupId>
      <artifactId>scalacl</artifactId>
      <version>0.2</version>
    </dependency>
  </dependencies>

  <build>
    ...
    <plugins>
    	<plugin>
          <groupId>org.scala-tools</groupId>
          <artifactId>maven-scala-plugin</artifactId>
          <configuration>
            <compilerPlugins>
              <compilerPlugin>
                <groupId>com.nativelibs4java</groupId>
                <artifactId>scalacl-compiler-plugin</artifactId>
                <version>0.2</version>
                <!-- or "0.3-SNAPSHOT" to get the latest development version -->
              </compilerPlugin>
            </compilerPlugins>
          </configuration>
        </plugin>
    </plugins>
  </build>
...
```
If you choose the development version, Maven will automatically update the plugin to the latest available snapshot version each time you run it without the `-o` switch.

# With Eclipse #

## Configure the ScalaCL Compiler Plugin ##

Here's how to configure an [Eclipse Scala](http://www.scala-ide.org/) project so it uses the ScalaCL Compiler Plugin :
  * Download the [latest stable version of the ScalaCL Compiler Plugin](http://nativelibs4java.sourceforge.net/maven/com/nativelibs4java/scalacl-compiler-plugin/0.2/scalacl-compiler-plugin-0.2.jar) (or the [latest development version](http://oss.sonatype.org/content/groups/public/com/nativelibs4java/scalacl-compiler-plugin/0.3-SNAPSHOT/))
> Please not that **this is not an update site**, but an URL meant for manual download of ScalaCL's jars.
  * In case you downloaded a development version, rename the timestamped snapshot file exactly to `scalacl-compiler-plugin-0.3-SNAPSHOT.jar` (i.e. replace the timestamp by `SNAPSHOT`), otherwise the compiler plugin mechanism won't work :-S
  * Open the section "Scala Compiler Properties" of Eclipse project's properties
  * Check "Use Project Settings" to enable the configuration items below
  * Fill the `Xplugin` field with the full path to the JAR you've downloaded, for instance :
```
/Users/ochafik/ScalaCLPlugin/target/scalacl-compiler-plugin-xxx.jar
```
  * Fill the `Xplugin-require` field with `scalacl` (just in case)
  * Clean your project, so it is rebuilt with the plugin enabled (menu `Project / Clean...`)
In case of issue, it is advised you compile using Maven or scalac+sbaz, because it seems the output from the compiler and from the compiler plugin is not visible from Eclipse.

# Disable the ScalaCL Compiler Plugin #

## Global disable ##

The plugin is enabled by default. To disable it, set the environment variable DISABLE\_SCALACL\_PLUGIN to 1 :
```
DISABLE_SCALACL_PLUGIN=1 scalac ...
```

## Selective disable (skip some files) ##

To disable optimizations for a whole file or at a specific line in a file, use the SCALACL\_SKIP environment variable :
```
SCALACL_SKIP=file1,file2.scala,../path/to/file3.scala,file4:someLine,file5:someLine...
```
(this can be used to troubleshoot the ScalaCL Compiler Plugin : you can narrow compiler crashes down to the faulty line(s))