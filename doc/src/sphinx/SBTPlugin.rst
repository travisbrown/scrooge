SBT Plugin
==========

Add a line like this to your `project/plugins.sbt` file:

::

    addSbtPlugin("com.twitter" %% "scrooge-sbt-plugin" % "3.18.0")

In your `build.sbt` file:

::

    libraryDependencies ++= Seq(
      "org.apache.thrift" % "libthrift" % "0.8.0",
      "com.twitter" %% "scrooge-core" % "3.18.0",
      "com.twitter" %% "finagle-thrift" % "6.24.0"
    )

or, in your `project/Build.scala` file:

::

    lazy val myProject = Project(
      id = "my-project",
      base = file("my-project"),
      settings = Project.defaultSettings
    ).settings(
      libraryDependencies ++= Seq(
        "org.apache.thrift" % "libthrift" % "0.8.0",
        "com.twitter" %% "scrooge-core" % "3.18.0",
        "com.twitter" %% "finagle-thrift" % "6.24.0"
      )
    )

**Configuration Options**

A full list of settings is in the (only) source file. Here are the ones you're
most likely to want to edit:

- **scroogeBuildOptions: Seq[String]**

  list of command-line arguments to pass to scrooge
  (default: `Seq("--finagle", "--verbose")`)

- **scroogeThriftDependencies: Seq[String]**

  artifacts to extract and compile thrift files from

- **scroogeThriftIncludeFolders: Seq[File]**

  list of folders to search when processing "include" directives
  (default: none)

- **scroogeThriftSourceFolder: File**

  where to find thrift files to compile
  (default: `src/main/thrift/`)

- **scroogeThriftOutputFolder: File**

  where to put the generated scala files
  (default: `target/<scala-ver>/src_managed`)


Migrating from 3.x.x to 3.18.0
------------------------------

Sbt 0.13.5 and above
~~~~~~~~~~~~~~~~~~~~

Autoplugins are available only for sbt 0.13.5 and above.


Projects using  `build.sbt`
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    import com.twitter.scrooge.ScroogeSBT

    lazy val app = project.in(file("app"))
      .settings(ScroogeSBT.newSettings: _*)
      .settings(
        // more settings here
      )

It now should look like

::

    lazy val app = project.in(file("app"))
      .settings(
        // more settings here
      )


Settings are included automatically, by virtue of the autoplugin mechanism for all JvmPlugin projects.

If you use a setting, this is in scope automatically as well so

::

    import com.twitter.scrooge.ScroogeSBT

    lazy val app = project.in(file("app"))
      .settings(ScroogeSBT.newSettings: _*)
      .settings(
        ScroogeSBT.scroogeThriftSourceFolder in Compile <<= baseDirectory {
          base => base / "src/main/resources"
        }
      )

becomes

::

    lazy val app = project.in(file("app"))
      .settings(
        scroogeThriftSourceFolder in Compile <<= baseDirectory {
          base => base / "src/main/resources"
        }
      )


Using `Build.scala`
~~~~~~~~~~~~~~~~~~~

The big change here, as well as not needing to inject settings, is that the location of the keys has changed. They are now under an object called autoImport.

::

    import sbt._
    import Keys._

    import com.twitter.scrooge.ScroogeSBT._

    object build extends Build {
      lazy val app = Project(
        id = "app",
        settings = Project.defaultSettings ++ newSettings
      ).settings(
        scroogeThriftSourceFolder in Compile <<= baseDirectory {
          base => base / "src/main/resources"
        }
      )
    }

becomes

::

    import sbt._
    import Keys._

    import com.twitter.scrooge.ScroogeSBT.autoImport._

    object build extends Build {

      lazy val app = Project(
        id = "app",
        base = file("app"),
        settings = Project.defaultSettings
      ).settings(
        scroogeThriftSourceFolder in Compile <<= baseDirectory {
          base => base / "src/main/resources"
        }
      )
    }

That is to say: adjust the imports, and drop the settings injection.
