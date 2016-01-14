Groovy 安装
==========




1. Download
-------------

In this download area, you will be able to download the distribution (binary and source), the Windows installer and the documentation for Groovy.

For a quick and effortless start on Mac OSX, Linux or Cygwin, you can use GVM (the Groovy enVironment Manager) to download and configure any Groovy version of your choice. Basic instructions can be found below.

1.1. Stable
~~~~~~~~~~~~~~

Download zip: Binary Release | Source Release

Download documentation: JavaDoc and zipped online documentation

Combined binary / source / documentation bundle: Distribution bundle

You can learn more about this version in the release notes or in the changelog.

If you plan on using invokedynamic support, read those notes.

1.2. Snapshots
~~~~~~~~~~~~~~

For those who want to test the very latest versions of Groovy and live on the bleeding edge, you can use our snapshot builds. As soon as a build succeeds on our continuous integration server a snapshot is deployed to Artifactory’s OSS snapshot repository.

1.3. Prerequisites
~~~~~~~~~~~~~~

Groovy 2.4 requires Java 6+ with full support up to Java 8. There are currently some known issues for some aspects when using Java 9 snapshots. The groovy-nio module requires Java 7+. Using Groovy’s invokeDynamic features require Java 7+ but we recommend Java 8.

The Groovy CI server is also useful to look at to confirm supported Java versions for different Groovy releases. The test suite (getting close to 10000 tests) runs for the currently supported streams of Groovy across all the main versions of Java each stream supports.

2. Maven Repository
--------------------------

If you wish to embed Groovy in your application, you may just prefer to point to your favourite maven repositories or the JCenter maven repository.

2.1. Stable Release
~~~~~~~~~~~~~~~~~~~~~~~~~

Gradle	Maven	Explanation
'org.codehaus.groovy:groovy:2.4.5'

<groupId>org.codehaus.groovy</groupId> <artifactId>groovy</artifactId> <version>2.4.5</version>

Just the core of groovy without the modules (see below).

'org.codehaus.groovy:groovy-$module:2.4.5'

<groupId>org.codehaus.groovy</groupId> <artifactId>groovy-$module</artifactId> <version>2.4.5</version>

"$module" stands for the different optional groovy modules "ant", "bsf", "console", "docgenerator", "groovydoc", "groovysh", "jmx", "json", "jsr223", "servlet", "sql", "swing", "test", "testng" and "xml". Example: <artifactId>groovy-sql</artifactId>

'org.codehaus.groovy:groovy-all:2.4.5'

<groupId>org.codehaus.groovy</groupId> <artifactId>groovy-all</artifactId> <version>2.4.5</version>

The core plus all the modules. Optional dependencies are marked as optional. You may need to include some of the optional dependencies to use some features of Groovy, e.g. AntBuilder, GroovyMBeans, etc.

To use the InvokeDynamic version of the jars just append ':indy' for Gradle or <classifier>indy</classifier> for Maven.

3. GVM (the Groovy enVironment Manager)
-----------------------------------------

This tool makes installing Groovy on any Bash platform (Mac OSX, Linux, Cygwin, Solaris or FreeBSD) very easy.

Simply open a new terminal and enter:

.. code-block:: sh

	$ curl -s get.gvmtool.net | bash

Follow the instructions on-screen to complete installation.

Open a new terminal or type the command:

.. code-block:: sh

	$ source "$HOME/.gvm/bin/gvm-init.sh"

Then install the latest stable Groovy:

.. code-block:: sh

	$ gvm install groovy

After installation is complete and you’ve made it your default version, test it with:

.. code-block:: sh

	$ groovy -version

That’s all there is to it!

4. Other ways to get Groovy
-------------------------------

4.1. Installation on Mac OS X
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
4.1.1. MacPorts
^^^^^^^^^^^^^^^^^^^^^

If you’re on MacOS and have MacPorts installed, you can run:

.. code-block:: sh
	
	sudo port install groovy

4.1.2. Homebrew
^^^^^^^^^^^^^^^^^^^^^

If you’re on MacOS and have Homebrew installed, you can run:

brew install groovy

4.2. Installation on Windows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you’re on Windows, you can also use the NSIS Windows installer.

4.3. Other Distributions

You may download other distributions of Groovy from this site.

4.4. Source Code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you prefer to live on the bleeding edge, you can also grab the source code from GitHub.

4.5. IDE plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are an IDE user, you can just grab the latest IDE plugin and follow the plugin installation instructions.

5. Install Binary
--------------------

These instructions describe how to install a binary distribution of Groovy.

First, Download a binary distribution of Groovy and unpack it into some file on your local file system.

Set your ``GROOVY_HOME`` environment variable to the directory you unpacked the distribution.

Add ``GROOVY_HOME/bin`` to your PATH environment variable.

Set your JAVA_HOME environment variable to point to your JDK. On OS X this is ``/Library/Java/Home``, on other unixes its often /usr/java etc. If you’ve already installed tools like Ant or Maven you’ve probably already done this step.

You should now have Groovy installed properly. You can test this by typing the following in a command shell:

groovysh
Which should create an interactive groovy shell where you can type Groovy statements. Or to run the Swing interactive console type:

groovyConsole
To run a specific Groovy script type:

groovy SomeScript