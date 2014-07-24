# Launching JES

Launching JES is a somewhat involved process. While it boils down to
"launch the `JESstartup` class," many Java system properties need to be set
in order for Jython and JES to find everything they need.
So, JES has several launcher scripts that set up the necessary environment
before booting Java.


## Standard Launchers

Right now, there are three:

* The Windows generic launcher (`jes.bat` in the repository base)
* The Linux/Mac OS X generic launcher (`jes.sh` in the repository base)
* The Mac OS X .app launcher (`releases/resources/macosx/jes-launcher.sh`)

The two generic launchers assume that they live in the same directory
as the `jes` and `dependencies` directories. This means they can launch JES
directly from a source code checkout (after you build, of course!), and the
release process makes sure that Windows and Linux builds have a compatible
structure.

(As an aside, the `JES.exe` file included in the Windows build just calls
`jes.bat` and exits.)

The .app launcher is only used in `JES.app` bundles for OS X. It operates
slightly differently than the rest because it has to work with the `.app`
directory structure. (Specifically, `jes` and `dependencies` live in
the bundle's `Contents/Resources/Java` instead of right next to the launcher.)

All three of them allow arguments to be passed, which will be forwarded on to
the `JESstartup` class.


### Customizing Java

All of the launchers assume that Java is installed system-wide,
except for the Windows launcher, which looks for a JRE in
`dependencies/jre-win32`. You can tell the launchers to use a specific JRE
by setting the `JES_JAVA_HOME` environment variable.

(The system `JAVA_HOME` environment variable will also be consulted,
but `JES_JAVA_HOME` will only apply to JES, *and* it will override the
bundled JRE on Windows if necessary.)

By default, JES will only allocate 512 MB of heap. This is controlled by the
`JES_JAVA_MEMORY` environment variable, which contains JVM memory options
(`-Xmx512m` by default.) You can override this.

If you need to define system properties or otherwise customize Java,
you can provide additional options to pass to Java using the
`JES_JAVA_OPTIONS` environment variable.


## Debugging Options for JESstartup

There are a couple of options you can pass to the JESstartup class,
or the standard launchers.

* Pass `--properties` to print all the system properties
  instead of starting JES.
* Pass `--shell` to open a Python interactive prompt in your
  console instead of starting JES.
* Pass `--debug-keys` to print debugging information each keypress
  as it is delivered to Java.
* Pass `--check-threads` to print a stack trace whenever a thread other than
  the Event Dispatch Thread causes the GUI to change.
  (This won't catch all threading bugs, but it will catch many of them.)
* Pass `--python-verbose=<level>` to set the Jython verbosity level.
  This can be `error`, `warning`, `message`, `comment`, or `debug`
  (for example, `--python-verbose=comment`).


## The Environment Created by the Launchers

In order for JES to find what it needs, the launchers set a large number
of Java system properties and other things. You set a Java
property on the command line by passing an option that looks like:

    -Dproperty.name=property.value

before you pass the class name (`JESstartup`).


### Java Classpath (-cp)

The Java classpath controls where Java loads classes from.
JES contains a bunch of Java classes, whose source code lives in the
`jes/java` folder. It also contains a few JAR's, which are ZIP files full
of compiled classes. All of JES's JAR's were written by other people,
and they live in `dependencies/jars`.

When launching JES, all the JAR's need to be on the classpath, and so do
all of the JES classes.

The classpath is different from other properties, because it's set using a
`-cp` option, like:

    -cp one.jar:another.jar:classes

The generic launchers both assume that the compiled JES classes live in the
`jes/classes` directory, but the OS X .app launcher assumes that they have
been assembled into a `.jar` file, living in `jes/classes.jar`.


### Python Home/Path (python.home and python.path)

These properties control where Jython finds `.py` files to load.
JES's own Python code lives in `jes/python`, but it also needs to load the
Python standard library, because Jython's JAR file doesn't contain the
library.

So, `python.home` needs to be set to the directory that the bundled copy of
Jython lives in, and `python.path` needs to be set to the directory that
contains JES's Python code.


### Python Cache Directory (python.cachedir)

Jython uses this directory to store information about where all the Java
packages live. If you don't have this set, it uses a directory in
`dependencies/jython` instead, which could cause permission issues.
It's probably best to put the location in the user's home directory, in the
place suggested by the operating system.

Warning: If you set `python.cachedir` to a directory that doesn't exist,
Jython will use `dependencies/jython/cachedir` anyway. Make sure the directory
exists before you use it.


### JES Home (jes.home)

This property is used by JES itself to find its files.
(Specifically, the images it uses in its UI, and its help files.)
It will look for `python`, `images`, `help`, and `javadoc` subdirectories,
so the easiest thing to do is use the `jes` directory, which the
standard launchers do.


### JES Config File (jes.configfile)

This property is the path to the JES configuration file, including its
name (which is usually `JESConfig.properties`). This is where JES keeps all
the user settings which need to remain across restarts.

Most of the time users don't really need to see it or edit it.
So you should store it somewhere "behind the scenes" in the user's
home directory. (If it's left out, JES will put it *directly*
in the user's home directory, which may annoy people. I know it annoys me.)
