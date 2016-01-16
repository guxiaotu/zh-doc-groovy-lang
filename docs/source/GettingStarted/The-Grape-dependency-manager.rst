Grape 依赖管理器
================

入门
-----

添加依赖
^^^^^^^^

Grape 是 Groovy 集成的 JAR 包依赖管理器。Grape 可以让你快速添加 maven 仓库依赖到你的 classpath 中。
添加方式非常简易，加入一段注释到你的代码上，如：

.. code-block:: groovy

    @Grab(group='org.springframework', module='spring-orm', version='3.2.5.RELEASE')
    import org.springframework.jdbc.core.JdbcTemplate

也可以如下简化：

.. code-block:: groovy

    @Grab('org.springframework:spring-orm:3.2.5.RELEASE')
    import org.springframework.jdbc.core.JdbcTemplate

你可以使用以上推荐方式，采用注解形式引入依赖。也可以在 `mvnrepository.com <mvnrepository.com>`_ 查询依赖关系，其中会提供 ``@Grab`` 的 ``pom.xml`` 格式。

指定扩展仓库
^^^^^^^^^^^^^

并非所有依赖都在 maven central 中，可以通过以下方式添加指定扩展仓库:

.. code-block:: groovy


    @GrabResolver(name='restlet', root='http://maven.restlet.org/')
    @Grab(group='org.restlet', module='org.restlet', version='1.1.6')

Maven Classifiers
^^^^^^^^^^^^^^^^^

一些 maven 依赖需要进一步细分类别，可以通过如下方式定义：

.. code-block:: groovy

    @Grab(group='net.sf.json-lib', module='json-lib', version='2.2.3', classifier='jdk15')

清除传递依赖
^^^^^^^^^^^^^

有时考虑到版本兼容等问题，需要清除一部分传递依赖，可以做如下配置：

.. code-block:: groovy

    @Grab('net.sourceforge.htmlunit:htmlunit:2.8')
    @GrabExclude('xml-apis:xml-apis')

JDBC Drivers
^^^^^^^^^^^^

.. caution:: 

    下面这一段还没有理解清楚:

    Because of the way JDBC drivers are loaded, you’ll need to configure Grape to attach JDBC driver dependencies to the system class loader. I.e:

.. code-block:: groovy

    @GrabConfig(systemClassLoader=true)
    @Grab(group='mysql', module='mysql-connector-java', version='5.1.6')


在 Groovy Shell 中使用 Grape
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: groovy

    groovy.grape.Grape.grab(group:'org.springframework', module:'spring', version:'2.5.6')

Proxy 设置
^^^^^^^^^^

如果需要通过代理服务，可以使用一下命令进行配置：

.. code-block:: Shell

    groovy -Dhttp.proxyHost=yourproxy -Dhttp.proxyPort=8080 yourscript.groovy

如果需要在全局范围添加代理服务，需要增加 JAVA_OPTS 环境变量设置:

.. code-block:: Shell

    JAVA_OPTS = -Dhttp.proxyHost=yourproxy -Dhttp.proxyPort=8080

Logging
^^^^^^^

如果你需要看到 Grape 执行过程日志，可以将参数 groovy.grape.report.downloads 设置为 true （e.g. add -Dgroovy.grape.report.downloads=true to invocation or JAVA_OPTS），这样 Grape 将会打印如下信息到 System.error:

- Starting resolve of a dependency
- Starting download of an artifact
- Retrying download of an artifact
- Download size and time for downloaded artifacts

如果需要打印更过日志，可以调整 Ivy 日志级别 （defaults to -1）。
如： -Divy.message.logger.level=4.


Detail
------

Grape (The Groovy Adaptable Packaging Engine or Groovy Advanced Packaging Engine) 是 grab() 执行的基础环境。
它能允许开发人员通过脚本获取所需要的库。Grape 在运行期间，将会从 JCenter, Ibiblio and java.net 下载所需的库文件，并连接相应库，建立所有依赖的传递关系。

下载的模块将依照 Ivy 的标准规范缓存至 ``~/.groovy/grape`` 目录中。

使用方式
--------

Annotation 注解
^^^^^^^^^^^^^^^^^

groovy.lang.Grab 注解可以添加一个或多个在任意位置，注解内容将通知编译器代码依赖的库。这将会把库添加至 Groovy 编译器的类加载器中。注解的检查将先于类的解析

.. code-block:: groovy

    import com.jidesoft.swing.JideSplitButton
    @Grab(group='com.jidesoft', module='jide-oss', version='[2.2.1,2.3.0)')
    public class TestClassAnnotation {
        public static String testMethod () {
            return JideSplitButton.class.name
        }
    }

An appropriate grab(…​) call will be added to the static initializer of the class of the containing class (or script class in the case of an annotated script element).

Multiple Grape Annotations
^^^^^^^^^^^^^^^^^^^^^^^^^^

在同一处使用多个 Grape 需要使用 @Grapes 注解，如下:

.. code-block:: groovy

    @Grapes([
       @Grab(group='commons-primitives', module='commons-primitives', version='1.0'),
       @Grab(group='org.ccil.cowan.tagsoup', module='tagsoup', version='0.9.7')])
    class Example {
    // ...
    }

否则会遇到下面的错误提示：

	Cannot specify duplicate annotation on the same member

方法调用
^^^^^^^^

通常情况下 grab 执行会先于脚本或类的初始化。这是需要确保代码所依赖的库，已经加载进入类加载器中。
几种典型的调用方式：

.. code-block:: groovy

    import groovy.grape.Grape
    // random maven library
    Grape.grab(group:'com.jidesoft', module:'jide-oss', version:'[2.2.0,)')
    Grape.grab([group:'org.apache.ivy', module:'ivy', version:'2.0.0-beta1', conf:['default', 'optional']],
         [group:'org.apache.ant', module:'ant', version:'1.7.0'])

- 在同一上下文（context）中使用相同参数，调用 grab 多次，需要保证幂等性。相同代码如果在不同的类加载器中调用，需要被再次执行。
- 如果 `args` 中传递给 grab 的属性中包括 noExceptions 这将不再有异常抛出。
- grab 中的 RootLoader 或 GroovyClassLoader 需要指定或着是在调用类的类加载器链中。默认情况下以上类加载器获取失败，将会抛出异常。


命令行工具
------------

Grape 添加命令 ``grape`` 用于检查管理本地 grape 缓存

.. code-block:: Shell

    grape install <groupId> <artifactId> [<version>]

这将会安装指定的 groovy module 或 maven artifact。如果 version 指定将安装指定版本，否则安装最新版本。


.. code-block:: Shell

    grape list


列出已经安装的 modules 及其版本号。

.. code-block:: Shell

    grape resolve (<groupId> <artifactId> <version>)+

以上返回 module 文件路径

高级配置
--------

资源库路径
^^^^^^^^^^


如果你需要改变 grape 默认的下载路径，你需要修改 grape.root 配置项默认值(default: ~/.groovy/grape)

.. code-block:: Shell

    groovy -Dgrape.root=/repo/grape yourscript.groovy

自定义 Ivy 设置
^^^^^^^^^^^^^^^

你可以创建 ~/.groovy/grapeConfig.xml 用于自定义 Ivy 配置。
如果没有此文件，`这里 <https://github.com/apache/incubator-groovy/blob/master/src/resources/groovy/grape/defaultGrapeConfig.xml>`_ 可以是 Grape 默认设置。

如何自定义 Ivy 设置更多的信息，可以参考 `Ivy 文档 <https://ant.apache.org/ivy/history/latest-milestone/index.html>`_


More Examples
-------------


.Using Apache Commons Collections:

.. code-block:: groovy

    // create and use a primitive array list
    import org.apache.commons.collections.primitives.ArrayIntList

    @Grab(group='commons-primitives', module='commons-primitives', version='1.0')
    def createEmptyInts() { new ArrayIntList() }

    def ints = createEmptyInts()
    ints.add(0, 42)
    assert ints.size() == 1
    assert ints.get(0) == 42


.Using TagSoup:

.. code-block:: groovy

    // find the PDF links of the Java specifications
    @Grab(group='org.ccil.cowan.tagsoup', module='tagsoup', version='1.2.1')
    def getHtml() {
        def parser = new XmlParser(new org.ccil.cowan.tagsoup.Parser())
        parser.parse("https://docs.oracle.com/javase/specs/")
    }
    html.body.'**'.a.@href.grep(~/.*\.pdf/).each{ println it }

.Using Google Collections:

.. code-block:: groovy

    import com.google.common.collect.HashBiMap
    @Grab(group='com.google.code.google-collections', module='google-collect', version='snapshot-20080530')
    def getFruit() { [grape:'purple', lemon:'yellow', orange:'orange'] as HashBiMap }
    assert fruit.lemon == 'yellow'
    assert fruit.inverse().yellow == 'lemon'


.Launching a Jetty server to serve Groovy templates:

.. code-block:: groovy

    @Grapes([
        @Grab(group='org.eclipse.jetty.aggregate', module='jetty-server', version='8.1.7.v20120910'),
        @Grab(group='org.eclipse.jetty.aggregate', module='jetty-servlet', version='8.1.7.v20120910'),
        @Grab(group='javax.servlet', module='javax.servlet-api', version='3.0.1')])

    import org.eclipse.jetty.server.Server
    import org.eclipse.jetty.servlet.*
    import groovy.servlet.*

    def runServer(duration) {
        def server = new Server(8080)
        def context = new ServletContextHandler(server, "/", ServletContextHandler.SESSIONS);
        context.resourceBase = "."
        context.addServlet(TemplateServlet, "*.gsp")
        server.start()
        sleep duration
        server.stop()
    }

    runServer(10000)


Grape 将下载 Jetty 和其他相关的依赖包，在第一次启动脚本时，并且缓存他们。
我们创建了一个 Jetty Server 并且监听 8080 端口，并发布  Groovy’s TemplateServlet 在根路径上。
Groovy 使用它自身强大模版引擎。我们启动服务，并让服务运行固定时间。
当访问 http://localhost:8080/somepage.gsp ， 页面将显示 somepage.gsp 模版内容给用户。
这些模版页面必须位于服务脚本的相同目录下。





