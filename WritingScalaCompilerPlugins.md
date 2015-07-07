# Introduction #

## Tools ##

  * Netbeans 6.9.1 + latest Scala Plugin = stable and feature-full IDE (go to definition, in-place renaming, auto-completion that really works all the time, pretty decent debugger that actually breaks where you want), and it works great with Maven projects.

## Documentation ##

  * The [Scala Compiler Corner](http://www.sts.tu-harburg.de/people/mi.garcia/ScalaCompilerCorner/) (and its [reloaded](http://lamp.epfl.ch/~magarcia/ScalaCompilerCornerReloaded/) version)
  * [Documentation](https://docs.google.com/viewer?url=http://scala-refactoring.org/wp-content/uploads/scala-refactoring.pdf) of [Mirko Stocker](http://misto.ch/)'s [Scala Refactoring Project](http://scala-refactoring.org/). **READ THIS OR YOU'LL BE COMPLETELY LOST !**
  * [TypeSymbol vs. TermSymbol](http://scala-programming-language.1934581.n4.nabble.com/Compiler-plugin-to-turn-Scala-into-Closure-annotated-Javascript-td2952844.html)
  * [three videos about compiler internals](http://www.scala-lang.org/node/598)

TODO more links

## Workflow ##

For each transformation wanted, write two code snippets :
  * `Code1.scala` : the code before the transform
  * `Code2.scala` : the equivalent code after the transform

Then run scalac on each of these two samples twice with the following options :
  * `scalac -Xprint:typer Code.scala` will give you a human-friendly look at how your code looks like after the `typer` phase. It's still Scala, but some transformations already took place (for loops, int ranges...)
  * `scalac -Xprint:typer -Yshow-trees Code.scala` will print the same code with composed extractors (modulo some missing parenthesis) : it will be your basic material for pattern matching (equivalent code snippet = `nodeToString(tree)`)
  * `scalac -Xbrowse:typer Code.scala` will let you explore the AST in a Swing GUI (equivalent code snippet = `treeBrowsers.create.browse(tree)`)

TODO more