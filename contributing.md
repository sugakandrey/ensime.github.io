---
layout: page
order: 10
title: Contributing
site_nav_entry: true # this is an entry in the main site nav
---

- TOC
{:toc}

The general architecture of ENSIME looks like this:

![architecture](/talks/scalasphere16/images/architecture.png)

which is explained in a recent [talk about ENSIME](/talks/scalasphere16/) by @rorygraves and @fommil. All our code is hosted at [github.com/ensime](https://github.com/ensime/).

## Where to Start

The first thing to do is to decide what you want to contribute!

If you are looking for a small task to pick up, have a look at our [waffle board](https://waffle.io/ensime/ensime-server). If that's too intimidating, restrict it to just the [2.0 Milestone](https://waffle.io/ensime/ensime-server?milestone=Big%20Crunch%202.0).

Anything under the **Low Hanging Fruit** or **Needs Reproduction** columns are suitable for a new contributor. Please contact us on the github issue directly if you want to contribute something: this avoids duplication of efforts, and also lets us discuss the best way to do it.

We have regular [Hack Days in London](http://hackthetower.co.uk/). If you can set aside the time to join us, they are a fun and exciting way to contribute to ENSIME.

If you can't come to a hack day but you'd really like to have some live help from somebody, please ask. We can use the libre software [meet.jit.si](https://meet.jit.si/) to screen share and discuss tickets anytime that is convenient.

If you'd like an ENSIME logo sticker for your laptop, let us know in your PR and privately email your postal address (or feel literally [free](http://www.gnu.org/philosophy/free-sw.en.html) to print your own, the logo is [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) and is not trademarked).


## Editors

If you want to work on editor support for ENSIME there may be additional documentation at these locations:

- [Emacs](/editors/emacs/contributing)
- [Atom](/editors/atom/contributing)
- [Visual Studio Code](/editors/vscode/contributing)
- [Vim](/editors/vim/contributing)
- [Sublime](/editors/sublime/contributing)

If you want to add support for an editor not listed here, please get in touch at [gitter.im/ensime/ensime-server](https://gitter.im/ensime/ensime-server) and read the server documentation below to get an idea of what the API looks like.


## Build Tools

If you want to work on build tool support, there may be documentation at these locations:

- [SBT](/build_tools/sbt#contributing)
- [Maven](/build_tools/maven#contributing)
- [Gradle](/build_tools/gradle#contributing)

If you want to add support for a new build tool, the [manual `.ensime`](/build_tools/manual) instructions are a good place to start.


## Scala Compiler and Refactoring

Much of the Scala support of ENSIME is provided by [the scala compiler itself](https://github.com/scala/scala) (see also the [Typelevel fork of scalac](https://github.com/typelevel/scala)) and the [refactoring library](https://github.com/scala-ide/scala-refactoring). Scala IDE also uses these components and their documentation on accessing the [presentation compiler](http://scala-ide.org/docs/dev/architecture/presentation-compiler.html#scalapresentationcompiler) is recommended reading. We also recommend watching [a recent talk about scala-refactoring](https://twitter.com/mlangc/status/697322490482315264) by @mlangc.

If you'd like to improve these components, you may wish to ask questions at [gitter.im/scala/contributors](https://gitter.im/scala/contributors), [gitter.im/typelevel/scala](https://gitter.im/typelevel/scala) and [gitter.im/scala-ide/scala-ide](https://gitter.im/scala-ide/scala-ide) respectively.

To hack on the scala compiler with ENSIME, you need to build a custom `.ensime` file by following the [scalac ENSIME guide](/contributing/scalac).

Many problems (e.g. the infamous ["red squiggles on valid code"](https://github.com/ensime/ensime-server/issues/673)) may be incredibly difficult to solve in the compiler and a workaround is being investigated in [github.com/fommil/imaginary-friend](https://github.com/fommil/imaginary-friend), which would also welcome contributors.


## Server

The remainder of this document focuses on contributing to the server component, which is hosted at [github.com/ensime/ensime-server](https://github.com/ensime/ensime-server). If you pick up a ticket, please comment on it to let us know so that we can help (and also to avoid overlapping with somebody else). Feel free to ask questions on the github issue tracker or [gitter.im/ensime/ensime-server](https://gitter.im/ensime/ensime-server).

The server API is documented in [org/ensime/api](https://github.com/ensime/ensime-server/tree/2.0/api/src/main/scala/org/ensime/api)
with example JSON payloads in [org/ensime/jerky](https://github.com/ensime/ensime-server/blob/2.0/protocol-jerky/src/test/scala/org/ensime/jerk/JerkFormatsSpec.scala). The preferred protocol for most editors is JSON over WebSockets, which we call our JERKY protocol. A legacy S-Expression over TCP protocol is used by Emacs (deriving from the SWANK protocol of [SLIME](https://github.com/slime/slime)), but ENSIME 2.0 will hopefully see us move to [S-Expressions over WebSockets](https://github.com/ensime/ensime-server/issues/1189), which we will call our SWANKY protocol.

### Compiling and Tests

Please make sure to install [sbt-extras](https://github.com/paulp/sbt-extras), otherwise project compilation will fail with a `java.lang.OutOfMemoryError` or `java.lang.StackOverflowError`. If you cannot install `sbt-extras` you must make sure to pass the flags in `.jvmopts` to your sbt process. Please vote for or fix [sbt#2161](https://github.com/sbt/sbt/issues/2161) to remove this requirement.

We use Java 6 in our CI because we are part of the [community builds](https://github.com/scala/community-builds), but you should be able to use Java 7 for local development. Many of our tests are making assertions on classpath searches and some of our assumptions (such as method signatures and orderings of Java classes) may not be true for recent versions of Java. In ENSIME 2.0 we will move to [Java 7 and Java 8](https://github.com/ensime/ensime-server/issues/1118) in our CI.

Before you start, run this SBT command on your `ensime-server` repository as the `.ensime` file is required to run the integration tests (even if you are not using ENSIME to hack on ENSIME).

```
sbt ensimeConfig
```

Don't forget to compile the integration tests as well as the tests, e.g.

```
sbt test:compile it:compile
```

The `.drone.yml` file documents the exact commands that we use during our CI and should serve as a good reference.

### Implementing a Feature or Bugfix

It all starts with a test. Find a test that is already testing something similar to what you want to achieve, and adapt it.

We have several styles of tests: unit tests, in-memory source tests (e.g. [`RichPresentationCompilerSpec`](https://github.com/ensime/ensime-server/blob/master/core/src/it/scala/org/ensime/core/RichPresentationCompilerSpec.scala)), on-disk source tests (e.g. [`BasicWorkflow`](https://github.com/ensime/ensime-server/blob/master/core/src/it/scala/org/ensime/intg/BasicWorkflow.scala)) and the Emacs client tests. It may be instructive for you to read and understand some of these tests to get a feel for the right level to write your test.

### `2.0-graph` Beta Tesing

Are you willing to help us bring new exciting features to ENSIME, or perhaps you enjoy using <s>unstable and bug-ridden</s>bleeding edge software? Then consider trying out `2.0-graph-SNAPSHOT` version of ENSIME server. `2.0-graph` branch of ENSIME attempts to bring features like reverse find usages and show implementing members, but it is yet far from finished. To try it out (requires Java 7+) you can either download assembly-jar from <http://ensime.typelevel.org/> (see [Manual QA Tesing](http://ensime.github.io/contributing/#manual-qa-testing) for direct instructions), or if you are an emacs user simply add `(setq ensime-server-version "2.0.0-graph-SNAPSHOT")` to your `init.el`, then add `-XX:MaxDirectMemorySize=4g` to your JVM options, (assuming you are an SBT user just drop `javaOptions += "-XX:MaxDirectMemorySize=4g"` in your `build.sbt` or `Build.scala`) and you are all set. If you decide to give it a try, remember we want your feedback! Please, leave any thoughts you have in form of issues at [github milestone](https://github.com/ensime/ensime-server/milestone/11) or get in touch in [ensime-server gitter room](https://gitter.im/ensime/ensime-server). **Attention!** `2.0-graph` **branch <s>is</s>may be unstable and is not meant for regular usage yet.**

### Guidelines

We do not have any official style guide. Code formatting is enforced by scalariform. When in doubt, prefer functional idioms encouraged by [Typelevel](http://typelevel.org) such as referential transparency, [typeclass derivation](https://github.com/fommil/shapeless-for-mortals) and immutable data structures. We made a huge mistake depending on akka and we would like to [fix that by replacing it](https://github.com/ensime/ensime-server/issues/1351).

Note that many functional styles of programming [introduce huge memory overheads](https://skillsmatter.com/skillscasts/6939-optimising-scala-for-fun-and-profit) and we have several critical paths where we cannot afford to trade style over performance. We encourage *locally scoped* mutability if it improves performance and / or readability. We integrate with several heavily mutable external systems, such as the ASM classpath visitor, file systems, databases and the compiler APIs, so we try to be pragmatic.

Because we are at the forefront of the development cycle for new versions of Scala, we prefer to restrict our usage of external scala libraries. Therefore, please prefer Java dependencies where possible.

### Manual QA Testing

If an `-assembly.jar` file exists in your `.emacs.d/ensime`, `.atom/packages/Ensime` or `.config/ensime-vim` directory (for the expected binary version of scala and ENSIME) then it will be used in preference to the `sbt` auto-update procedure. This is advantageous for developing on ENSIME and also to enable a simple install of the ENSIME server in restricted environments. SNAPSHOT assembly jars are provided at <http://ensime.typelevel.org/> (with many thanks to Typelevel for the use of their servers).

To build your own server jars, do this:

```
git clone https://github.com/ensime/ensime-server.git
sbt ++2.10.6 ensime/assembly # replace with your version of scala
cp target/scala-2.10/ensime_2.10-0.9.10-SNAPSHOT-assembly.jar ~/.emacs.d/ensime/
```

When you want to swap back to using official releases, delete your `-assembly.jar` files.

You can also `sbt publishLocal` but make sure you clean up your `~/.ivy2/local` afterwards.
