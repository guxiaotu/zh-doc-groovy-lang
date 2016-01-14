
Groovy 安装
==========




下载
-------------

在下载区，你可以下载 Groovy 的安装包，源文件，windows 安装文件以及文档。
为了能轻松快速的在 ``Mac OSX`` , ``Linux`` 以及 ``Cygwin`` 上开始使用，你可以使用 `GVM <http://gvmtool.net/>`_ （ Groovy 环境管理器），下载并配置你选择的 Groovy 版本。下面是关于其的基础说明。

稳定版本
~~~~~~~~~~~~~~

- 下载 zip：`Binary Release <https://bintray.com/artifact/download/groovy/maven/apache-groovy-binary-2.4.5.zip>`_ | `Source Release <https://bintray.com/artifact/download/groovy/maven/apache-groovy-src-2.4.5.zip>`_
- 下载文档：`JavaDoc and zipped online documentation <https://bintray.com/artifact/download/groovy/maven/apache-groovy-docs-2.4.5.zip>`_
- 安装包 & 源代码 & 文档 : `Distribution bundle <https://bintray.com/artifact/download/groovy/maven/apache-groovy-sdk-2.4.5.zip>`_

你可以从发布记录以及变更日志中了解此版本的更多信息。

如果你计划使用 ``invokedynamic`` ，可以阅读这些 `文档 <http://docs.groovy-lang.org/latest/html/documentation/invokedynamic-support.html>`_。

快照版本
~~~~~~~~~~~~~~
如果你希望体验 Groovy 中最新版本,可以使用快照版本 (snapshot) 构建。 快照版本在 CI 服务中构建成功就会立即发布到我们的 ``Artifactory’s OSS`` 快照仓库中。

前置条件
~~~~~~~~~~~~~~

Groovy 2.4 需要 Java 6 以上版本支持，完美支持需要升级至 Java 8。目前在使用 Java 9 的 snapshot 上还存在一些问题。``groovy-nio`` 模块需要 Java 7 以上版本支持。使用 ``invokeDynamic`` 的特性可以使 Java 7+，但还是建议使用 Java 8。

Groovy CI 服务对于 Java 版本对 Groovy 版本支持情况的检查非常有用。测试套件 （接近 10000 个测试用例）将鉴证当前主流 Java 对于 Groovy 的支持情况。

Maven Repository
--------------------------
如果你希望在应用中集成 Groovy , 你只需要使用你习惯的 ``maven repositories`` 或 `JCenter maven repository <https://oss.jfrog.org/oss-release-local/org/codehaus/groovy>`_。

稳定版本
~~~~~~~~~~~~~~~~~~~~~~~~~

.. table:: 列表
   :class: classic

+------------------------------------+-----------------------------------------+---------------------------------+
| Gradle                             | Maven                                   | Explanation                     |
+====================================+=========================================+=================================+
|`org.codehaus.groovy:groovy:2.4.5`  |  <groupId>org.codehaus.groovy</groupId> | Just the core                   |
|                                    |    <artifactId>groovy</artifactId>      | of groovy without               |
|                                    |    <version>2.4.5</version>             | the modules (see below)         | 
+------------------------------------+-----------------------------------------+---------------------------------+
| org.codehaus.groovy:groovy-        | <groupId>org.codehaus.groovy</groupId>  | Example:                        |
| $module:2.4.5`                     | <artifactId>groovy-$module</artifactId> |  <artifactId>groovy-sql         |
|                                    | <version>2.4.5</version>                |  </artifactId>                  | 
|                                    |                                         |                                 |
+------------------------------------+-----------------------------------------+---------------------------------+
|org.codehaus.groovy:groovy-all:2.4.5| <groupId>org.codehaus.groovy</groupId>  |                                 |
|                                    | <artifactId>groovy-all</artifactId>     |                                 |
|                                    | <version>2.4.5</version>                |                                 |
|                                    |                                         |                                 |
+------------------------------------+-----------------------------------------+---------------------------------+

使用 ``InvokeDynamic`` 的 jars ，在 Gradle 中添加 ``:indy`` 或在 Maven 中添加 ``<classifier>indy</classifier>``。



GVM (the Groovy enVironment Manager)
-----------------------------------------

使用 GVM 在 这些平台  (Mac OSX, Linux, Cygwin, Solaris or FreeBSD) 的 Bash 上安装 Groovy 都会非常的容易。

打开一个 ``terminal`` ，输入：

.. code-block:: sh

	$ curl -s get.gvmtool.net | bash

根据屏幕上的引导说明完成安装。

打开一个新的 ``terminal`` 或输入下面命令：

.. code-block:: sh

	$ source "$HOME/.gvm/bin/gvm-init.sh"

开始安装最新稳定版本的 Groovy：

.. code-block:: sh

	$ gvm install groovy

安装完毕后，此版本为当前的默认版本，可以输入以下命令测试：

.. code-block:: sh

	$ groovy -version

就是如此简单。

其他方式安装 Groovy
-------------------------------

 Mac OS X 上安装
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

MacPorts
^^^^^^^^^^^^^^^^^^^^^
在 ``MacOS`` 上已经安装 `MacPorts <http://www.macports.org/>`_ ，可以使用下面命令安装:

.. code-block:: sh
	
	sudo port install groovy

Homebrew
^^^^^^^^^^^^^^^^^^^^^
在 ``MacOS`` 上已经安装 `Homebrew <http://mxcl.github.com/homebrew>`_ ， 可以使用下面命令安装:

.. code-block:: sh

	brew install groovy

 Windows 上安装
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在 Windows 上可以使用 `NSIS Windows installer <http://docs.groovy-lang.org/latest/html/documentation/TODO-Windows+NSIS-Installer>`_.

Other Distributions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may download other distributions of Groovy from this site.

Source Code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果你更愿意尝鲜，也可以从 GitHub 上下载 `源代码 <https://github.com/apache/incubator-groovy>`_。

IDE plugin
--------------------
如果你使用 IDE , 你也可以使用最新的 IDE 插件，依照插件的安装步骤完成。

安装二进制包
--------------------

这里将介绍通过二进制包，安装 Groovy.

- 首先 `下载 <http://www.groovy-lang.org/install.html#download-groovy>`_ ，在文件系统将下载文件解压。

- 设置环境变量 ``GROOVY_HOME`` 为文件的解压目录。

- 将 ``GROOVY_HOME/bin`` 加入到 PATH 中。

- 设置 ``JAVA_HOME`` 环境变量指向你的 JDK。在 OS X 上为目录 ``/Library/Java/Home``，在其他 unix 系统中可能在
``/usr/java`` 目录中。如果你已经安装 Ant 或 Maven 等工具，可以忽略这一步。


现在你已经安装好了 Groovy ， 可以通过下面命令测试:

.. code-block:: sh

	groovysh	

这将会创建一个 Groovy 的交互 shell ，用于执行 Groovy 代码。也可以运行 `Swing interactive console <http://docs.groovy-lang.org/latest/html/documentation/tools-groovyconsole.html>`_ ,如：

.. code-block:: sh
	
	groovyConsole

运行一段 ``Groovy script`` 可以输入：

.. code-block:: sh

	groovy SomeScript


