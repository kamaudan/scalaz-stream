scalaz-stream
=============

[![Build Status](https://travis-ci.org/scalaz/scalaz-stream.svg?branch=master)](http://travis-ci.org/scalaz/scalaz-stream)
[![Gitter Chat](https://badges.gitter.im/scalaz/scalaz-stream.svg)](https://gitter.im/scalaz/scalaz-stream)
[![scaladoc](http://javadoc-badge.appspot.com/org.scalaz.stream/scalaz-stream_2.11.svg?label=scaladoc)](http://javadoc-badge.appspot.com/org.scalaz.stream/scalaz-stream_2.11)

### Where to get it ###
Scalaz stream were deprecated at sometime past 

To get the latest version of the library, add the following to your SBT build:

```
// available for Scala 2.10.5, 2.11.7, 2.12.0-M1, 2.12.0-M2
libraryDependencies += "org.scalaz.stream" %% "scalaz-stream" % "0.8"
```

As of version 0.8, scalaz-stream is solely published against scalaz 7.1.x.  The most recent build for 7.0.x is scalaz-stream 0.7.3.

If you were using a previous version of scalaz-stream, you may have a `resolvers` entry for the Scalaz Bintray repository.  This is no longer required, as scalaz-stream is now published to Maven Central.  It won't hurt you though.

### About the library ###

`scalaz-stream` is a streaming I/O library. The design goals are compositionality, expressiveness, resource safety, and speed. The design is meant to supersede or replace older iteratee or iteratee-style libraries. Here's a simple example of its use:

``` scala
import scalaz.stream._
import scalaz.concurrent.Task

val converter: Task[Unit] =
  io.linesR("testdata/fahrenheit.txt")
    .filter(s => !s.trim.isEmpty && !s.startsWith("//"))
    .map(line => fahrenheitToCelsius(line.toDouble).toString)
    .intersperse("\n")
    .pipe(text.utf8Encode)
    .to(io.fileChunkW("testdata/celsius.txt"))
    .run

// at the end of the universe...
val u: Unit = converter.run
```

This will construct a `Task`, `converter`, which reads lines incrementally from `testdata/fahrenheit.txt`, skipping blanklines and commented lines. It then parses temperatures in degrees fahrenheit, converts these to celsius, UTF-8 encodes the output and writes incrementally to `testdata/celsius.txt`, using constant memory. The input and output files will be closed in the event of normal termination or exceptions.

The library supports a number of other interesting use cases:

* _Zipping and merging of streams:_ A streaming computations may read from multiple sources in a streaming fashion, zipping or merging their elements using a arbitrary `Tee`. In general, clients have a great deal of flexibility in what sort of topologies they can define--source, sinks, and effectful channels are all first-class concepts in the library.
* _Dynamic resource allocation:_ A streaming computation may allocate resources dynamically (for instance, reading a list of files to process from a stream built off a network socket), and the library will ensure these resources get released in the event of normal termination or when errors occur.
* _Nondeterministic and concurrent processing:_ A computation may read from multiple input streams simultaneously, using whichever result comes back first, and a pipeline of transformation can allow for nondeterminism and queueing at each stage.
* _Streaming parsing (UPCOMING):_ A separate layer handles constructing streaming parsers, for instance, for streaming JSON, XML, or binary parsing. See [the roadmap](https://github.com/scalaz/scalaz-stream/wiki/Roadmap) for more information on this and other upcoming work.

### Documentation and getting help ###

There are examples (with commentary) in the test directory [`scalaz.stream.examples`](https://github.com/scalaz/scalaz-stream/tree/master/src/test/scala/scalaz/stream/examples). Also see [the wiki](https://github.com/scalaz/scalaz-stream/wiki) for more documentation. If you use `scalaz.stream`, you're strongly encouraged to submit additional examples and add to the wiki!

For questions about the library, use the [scalaz mailing list](https://groups.google.com/forum/#!forum/scalaz) or the [scalaz-stream tag on StackOverflow](http://stackoverflow.com/questions/tagged/scalaz-stream).

Blog posts and other external resources are listed on the [Additional Resources](https://github.com/scalaz/scalaz-stream/wiki/Additional-Resources) page.

The wiki maintains a list of [projects using scalaz-stream](https://github.com/scalaz/scalaz-stream/wiki/Projects-Using-scalaz-stream).

### Related projects ###

[Machines](https://github.com/ekmett/machines/) is a Haskell library with the same basic design as `scalaz-stream`, though some of the particulars differ. There is also [`scala-machines`](https://github.com/runarorama/scala-machines), which is an older, deprecated version of the basic design of `scalaz-stream`.

There are various other iteratee-style libraries for doing compositional, streaming I/O in Scala, notably the [`scalaz/iteratee`](https://github.com/scalaz/scalaz/tree/scalaz-seven/iteratee) package and [iteratees in Play](http://www.playframework.com/documentation/2.0/Iteratees).
