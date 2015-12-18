# sbt-checkstyle-plugin [![Build Status](https://travis-ci.org/etsy/sbt-checkstyle-plugin.svg?branch=master)](https://travis-ci.org/etsy/sbt-checkstyle-plugin)

This project provides an SBT 0.13+ plugin for running Checkstyle over
Java source files.  For more information about Checkstyle, see
[http://checkstyle.sourceforge.net/](http://checkstyle.sourceforge.net/)

This plugin uses version 6.13 of Checkstyle.

This is a fork of the sbt-code-quality project found
[here](https://github.com/corux/sbt-code-quality).

## Setup

Add the following lines to `project/plugins.sbt`:

```scala
addSbtPlugin("com.etsy" % "sbt-checkstyle-plugin" % "1.0.0")
```

sbt-checkstyle-plugin is an AutoPlugin, so there is no need to modify the `build.sbt` file to enable it.

If you want to modify any of the default settings, you should add the following import to `build.sbt`, however:

```scala
import com.etsy.sbt.checkstyle._
```

## Usage

You can run Checkstyle over your Java source files with the
`checkstyle` task.  You can run Checkstyle over your Java tests with
the `test:checkstyle` task.

The Checkstyle configuration file is `./checkstyle-config.xml` by
default.  This can be changed by setting the value of
`Checkstyle.configLocation`.  By default `test:checkstyle` uses the same
configuration file, but this can be changed by setting the value of
`Checkstyle.configLocation in Test`.

The Checkstyle report is output to `target/checkstyle-report.xml` by
default.  This can be changed by setting the value of
`Checkstyle.outputFile`.  `test:checkstyle` outputs to
`target/checkstyle-test-report.xml`, but this can be changed by
setting the value of `Checkstyle.outputFile in Test`.

To change the checkstyle configuration file set `Checkstyle.configLocation` in `build.sbt`:
```scala
Checkstyle.CheckstyleTasks.checkstyleConfigLocation := Checkstyle.CheckstyleConfig.File("checkstyle-config.xml")
```

You can also load remote configuration files by specifying a URL:
```scala
Checkstyle.configLocation :=
  Checkstyle.CheckstyleConfig.URL("https://raw.githubusercontent.com/checkstyle/checkstyle/master/config/checkstyle_checks.xml")
```

Or load configuration files from the classpath by specifying a resource name:
```scala
Checkstyle.configLocation := Checkstyle.CheckstyleConfig.File("com/etsy/checkstyle-config.xml")
```

### XSLT transformations

The `xsltTransformations` setting allows applying XSLT transformations to the XML report generated by Checkstyle.  For instance, this could be used to generate a more readable HTML report.  This setting takes values of `Option[Set[XSLTSettings]]`, so multiple transformations can be applied.

You can set `xsltTransformations` like so in `build.sbt`:
```scala
Checkstyle.xsltTransformations := {
  Some(Set(XSLTSettings(baseDirectory(_ / "checkstyle-noframes.xml").value, target(_ / "checkstyle-report.html").value)))
}
```

### Failing the build

You can control what severity of issues should break the build by setting the `checkstyleSeverityLevel` in your `build.sbt` as follows:
```scala
Checkstyle.severityLevel := Some(Checkstyle.CheckstyleSeverityLevel.Error)
```

Possible values are defined by the `CheckstyleSeverityLevel` enumeration. The default is `None`.

### Integration tests

If you want to run Checkstyle on your integration tests add the following to your `build.sbt`:
```scala
lazy val root = (project in file(".")).configs(IntegrationTest)

Defaults.itSettings

Checkstyle.checkstyleSettings ++ Seq(
  Checkstyle.configLocation := Checkstyle.CheckstyleConfig.File("my-checkstyle-config.xml"),
  Checkstyle.checkstyle       in IntegrationTest <<= Checkstyle.checkstyleTask(IntegrationTest),
  Checkstyle.outputFile in IntegrationTest <<= target(_ / "checkstyle-integration-test-report.xml")
)
```

You can then run the tasks `it:checkstyle` and `it:checkstyle-check`.

### Upgrading Checkstyle version

SBT Checkstyle plugin comes with a default Checkstyle version: currently, Checkstyle 6.13 is used by default.

Provided the new Checkstyle version is compatible, you can override the version used at runtime in your `project/plugins.sbt`:

```scala
dependencyOverrides += "com.puppycrawl.tools" % "checkstyle" % "6.13"
```
