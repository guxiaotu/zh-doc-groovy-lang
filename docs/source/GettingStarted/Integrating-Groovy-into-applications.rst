应用集成 Groovy
===============


Groovy 集成机制
-------------------------------

Groovy 语言提供了几种在运行时集成至应用程序中的方法，包括从最基础的代码执行，到集成缓存及自定义编译器。
本章节的实例使用 Groovy 编写，其同样集成机制也适用于 Java 。

Eval
~~~~~~~~~~~~

``groovy.util.Eval`` 类在运行时，动态执行 Groovy 最简单的方式。可以直接通过方法 ``me`` 实现调用：

.. code-block:: groovy

	import groovy.util.Eval

	assert Eval.me('33*3') == 99
	assert Eval.me('"foo".toUpperCase()') == 'FOO'

``Eval`` 支持接受参数的多变种执行：

.. code-block:: groovy

	assert Eval.x(4, '2*x') == 8               //<1>              
	assert Eval.me('k', 4, '2*k') == 8         //<2>
	assert Eval.xy(4, 5, 'x*y') == 20          //<3>
	assert Eval.xyz(4, 5, 6, 'x*y+z') == 26    //<4>

<1> 绑定参数命名为 x

<2> 自定义绑定参数命名为 k

<3> 两个绑定参数分别命名为 x 和 y

<4> 三个绑定参数分别命名为 x ， y 和 z

``Eval`` 可以执行简单脚本，对于复杂的脚本将无能为力：由于其无法缓存脚本，但并不意味着只能执行单行脚本。

GroovyShell
~~~~~~~~~~~~~~~~~~

Multiple sources
^^^^^^^^^^^^^^^^^^^^^^^^^

``groovy.lang.GroovyShell`` 适用于需要缓存脚本实例的执行过程。
``Eval`` 类可以返回脚本执行结果， ``GroovyShell`` 类可以提供更多选择。

.. code-block:: groovy

	def shell = new GroovyShell()             					//<1>              
	def result = shell.evaluate '3*5'                       	//<2>
	def result2 = shell.evaluate(new StringReader('3*5'))   	//<3>
	assert result == result2
	def script = shell.parse '3*5'                              //<4>
	assert script instanceof groovy.lang.Script
	assert script.run() == 15    								//<5>

<1> 创建一个 ``GroovyShell`` 实例

<2> 与 ``Eval`` 一样可以直接执行代码

<3> 可以从多个数据源读取数据 （String, Reader, File, InputStream）

<4> 可以延缓脚本执行。``parse`` 返回 ``Script`` 实例

<5> ``Script`` 定义了一个 ``run`` 方法

Sharing data between a script and the application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用 ``groovy.lang.Binding`` 可以在应用程序与脚本之间共享数据：

.. code-block:: groovy
	
	def sharedData = new Binding()          					//<1>                
	def shell = new GroovyShell(sharedData)                 	//<2>
	def now = new Date()
	sharedData.setProperty('text', 'I am shared data!')         //<3>
	sharedData.setProperty('date', now)                     	//<4>

	String result = shell.evaluate('"At $date, $text"')     	//<5>

	assert result == "At $now, I am shared data!"

<1> 创建 ``Binding`` 对象，用于存储共享数据

<2> 创建 ``GroovyShell`` 对象，使用共享数据

<3> 添加字符串数据共享绑定

<4> 添加一个 date 类型数据绑定(共享数据不局限于基本数据类型)

<5> 执行脚本

.. note::
	脚本中也将数据写入共享数据对象。

.. code-block:: groovy
	
	def sharedData = new Binding()        			//<1>                  
	def shell = new GroovyShell(sharedData)         //<2>        

	shell.evaluate('foo=123')                       //<3>        

	assert sharedData.getProperty('foo') == 123     //<4>

<1> 创建 ``Binding`` 对象

<2> 创建 ``GroovyShell`` 对象，使用共享数据

<3> 使用未声明的变量存储数据结果至共享数据对象中

<4> 读取共享结果数据

从脚本中写入的共享数据，必须为未声明的变量。像下面例子中，使用定义或明确类型将会失败，其原因是这样你就已定义了一个本地变量：

.. code-block:: groovy

	def sharedData = new Binding()
	def shell = new GroovyShell(sharedData)

	shell.evaluate('int foo=123')

	try {
	    assert sharedData.getProperty('foo')
	} catch (MissingPropertyException e) {
	    println "foo is defined as a local variable"
	}

在多线程环境中使用共享数据，需要更加的小心。``Binding`` 实例传递给 ``GroovyShell`` 是非线程安全的，其在所有脚本中共享。


It is possible to work around the shared instance of Binding by leveraging the Script instance which is returned by parse:

.. code-block:: groovy

	def shell = new GroovyShell()

	def b1 = new Binding(x:3)    				//<1>                   
	def b2 = new Binding(x:4)                   //<2>    
	def script = shell.parse('x = 2*x')
	script.binding = b1
	script.run()
	script.binding = b2
	script.run()
	assert b1.getProperty('x') == 6
	assert b2.getProperty('x') == 8
	assert b1 != b2

<1> will store the x variable inside b1

<2> will store the x variable inside b2

然而，你需要注意的是你仍旧在同一个共享的脚本实例上。如果有两个线程在同一个脚本上工作，这项技术将不可使用。在那种情况下，就必须创建两个独立的脚本实例:

.. code-block:: groovy

	def shell = new GroovyShell()

	def b1 = new Binding(x:3)
	def b2 = new Binding(x:4)
	def script1 = shell.parse('x = 2*x')     			//<1>       
	def script2 = shell.parse('x = 2*x')            	//<2>
	assert script1 != script2
	script1.binding = b1                            	//<3>
	script2.binding = b2                            	//<4>
	def t1 = Thread.start { script1.run() }         	//<5>
	def t2 = Thread.start { script2.run() }         	//<6>
	[t1,t2]*.join()                                 	//<7>
	assert b1.getProperty('x') == 6
	assert b2.getProperty('x') == 8
	assert b1 != b2

<1> 为 thread 1 创建 script 实例

<2> 为 thread 2 创建 script 实例

<3> 第一个共享对象分配给 script 1

<4> 第二个共享对象分配给 script 2

<5> 在 thread 1 中启动 script 1

<6> 在 thread 2 中启动 script 2

<7> 等待执行结束

如果你需要线程安全，建议直接使用 `GroovyClassLoader <http://www.groovy-lang.org/integrating.html#groovyclassloader>`_


Custom script class
^^^^^^^^^^^^^^^^^^^^^^^^
我们已经知道 ``parse`` 方法返回 ``groovy.lang.Script`` 实例，还能够通过自定义扩展 ``Script`` 类来增强 ``script`` 的处理能力，例如：

.. code-block:: groovy

	abstract class MyScript extends Script {
	    String name

	    String greet() {
	        "Hello, $name!"
	    }
	}

扩展类自定义了一个 ``name`` 属性以及 ``greet`` 方法。通过配置这个类可以按照 ``script`` 类方式使用。

.. code-block:: groovy

	import org.codehaus.groovy.control.CompilerConfiguration                        

	def config = new CompilerConfiguration()                                             // <1>
	config.scriptBaseClass = 'MyScript'                                                  // <2>

	def shell = new GroovyShell(this.class.classLoader, new Binding(), config)           // <3>
	def script = shell.parse('greet()')                                                  // <4>
	assert script instanceof MyScript
	script.setName('Michel')
	assert script.run() == 'Hello, Michel!'

<1> 创建 ``CompilerConfiguration`` 实例

<2> 指定 ``scripts`` 执行的基类名称为 ``MyScript``

<3> 创建 ``shell`` 是指定当前定义的编译器配置信息

<4> 此 ``script`` 可以访问 ``greet`` 方法

并不限于使用唯一的 ``scriptBaseClass`` 配置。可以使用任意多个编译器配置，甚至自定义。


GroovyClassLoader
~~~~~~~~~~~~~~~~~~~~~~~~

前面章节，我看到 ``GroovyShell`` 是用来执行 ``scripts`` 的一个简单工具。在其内部，它使用 ``groovy.lang.GroovyClassLoader`` 在运行时编译并加载 ``classes``.

通过使用 ``GroovyClassLoader`` 替代 ``GroovyShell`` ，你可以加载 ``classes`` 替代 ``scripts`` 实例。

.. code-block:: groovy

    
	import groovy.lang.GroovyClassLoader

	def gcl = new GroovyClassLoader()             												// <1>                              
	def clazz = gcl.parseClass('class Foo { void doIt() { println "ok" } }')    				// <2>
	assert clazz.name == 'Foo'                                                  				// <3>
	def o = clazz.newInstance()                                                 				// <4>
	o.doIt()                                                                    				// <5>


<1> create a new GroovyClassLoader
<2> parseClass will return an instance of Class
<3> you can check that the class which is returns is really the one defined in the script
<4> and you can create a new instance of the class, which is not a script
<5> then call any method on it

``GroovyClassLoader`` 中维护它创建的所有 ``classes`` ， 所有存储溢出会比较容易出现。尤其是， 当你执行两次相同的脚本，你将得到两个不同的类！

.. code-block:: groovy

	import groovy.lang.GroovyClassLoader

	def gcl = new GroovyClassLoader()								
	def clazz1 = gcl.parseClass('class Foo { }')                    // <1>            
	def clazz2 = gcl.parseClass('class Foo { }')                     // <2>           
	assert clazz1.name == 'Foo' 									// <3>	                                                
	assert clazz2.name == 'Foo' 										
	assert clazz1 != clazz2                                         // <4>            

<1> dynamically create a class named "Foo"
<2> create an identical looking class, using a separate parseClass call
<3> make sure both classes have the same name
<4> but they are actually different!

如果你希望获得相同的实例， 源数据必须为文件，例如：

.. code-block:: groovy

	def gcl = new GroovyClassLoader()
	def clazz1 = gcl.parseClass(file)               						// <1>                            
	def clazz2 = gcl.parseClass(new File(file.absolutePath))                // <2>    
	assert clazz1.name == 'Foo'                                             // <3>    
	assert clazz2.name == 'Foo'
	assert clazz1 == clazz2     											// <4>		

<1> parse a class from a File
<2> parse a class from a distinct file instance, but pointing to the same physical file
<3> make sure our classes have the same name
<4> but now, they are the same instance

使用文件作为数据源， ``GroovyClassLoader`` 可以缓存生成的 ``class`` 文件， 这样可以避免在运行时创建多个类从单一源中。
Using a File as input, the GroovyClassLoader is capable of caching the generated class file, which avoids creating multiple classes at runtime for the same source.

GroovyScriptEngine
------------------

``groovy.util.GroovyScriptEngine`` 依靠 ``script`` 重载和依赖，为应用的灵活扩展提供基础设施。``GroovyShell`` 关注独立的 ``Script`` ， ``GroovyClassLoader`` 用于动态编译及加载 ``Groovy`` 类， ``GroovyScriptEngine`` 在 ``GroovyClassLoader`` 之上建立一层用于处理 ``script`` 的依赖及重新加载。

为说明这点，我们将创建一个 ``script engine`` 并在无限循环中执行代码。
首先创建下面代码：

ReloadingTest.groovy
++++++++++++++++++++

.. code-block:: groovy

	class Greeter {
	    String sayHello() {
	        def greet = "Hello, world!"
	        greet
	    }
	}

new Greeter()

然后使用 ``GroovyScriptEngine`` 执行代码：

.. code-block:: groovy

	def binding = new Binding()
	def engine = new GroovyScriptEngine([tmpDir.toURI().toURL()] as URL[])             // <1>     

	while (true) {
	    def greeter = engine.run('ReloadingTest.groovy', binding)                     // <2>
	    println greeter.sayHello()                                                    // <3>
	    Thread.sleep(1000)
	}

<1> create a script engine which will look for sources into our source directory
<2> execute the script, which will return an instance of Greeter
<3> print the greeting message

每秒钟将看到下面的打印信息：

|Hello, world!
|Hello, world!
|...

在不终止 ``script`` 执行，修改 ``ReloadingTest`` 文件，如：

.. code-block:: groovy

	//ReloadingTest.groovy
	class Greeter {
	    String sayHello() {
	        def greet = "Hello, Groovy!"
	        greet
	    }
	}

	new Greeter()

打印内容将会变化：	

|Hello, world!
|...
|Hello, Groovy!
|Hello, Groovy!
|...

下面将演示 ``script`` 的依赖，新建下面文件，同样不中断 ``script`` 的执行：

.. code-block:: groovy

	//Depencency.groovy
	class Dependency {
	    String message = 'Hello, dependency 1'
	}

更新 ``ReloadingTest``  文件:

.. code-block:: groovy

	//ReloadingTest.groovy
	import Dependency

	class Greeter {
	    String sayHello() {
	        def greet = new Dependency().message
	        greet
	    }
	}

	new Greeter()

打印内容将会有如下变化：	

｜Hello, Groovy!
｜...
｜Hello, dependency 1!
｜Hello, dependency 1!
｜...

你可以更新 ``Dependency.groovy`` 文件：

.. code-block:: groovy

	//Depencency.groovy
	class Dependency {
	    String message = 'Hello, dependency 2'
	}

你将看到 ``dependency`` 文件被从新加载：

｜Hello, dependency 1!
｜...
｜Hello, dependency 2!
｜Hello, dependency 2!

CompilationUnit
---------------

Ultimately, it is possible to perform more operations during compilation by relying directly on the org.codehaus.groovy.control.CompilationUnit class. This class is responsible for determining the various steps of compilation and would let you introduce new steps or even stop compilation at various phases. This is for example how stub generation is done, for the joint compiler.

However, overriding CompilationUnit is not recommended and should only be done if no other standard solution works.

Bean Scripting Framework
========================
2. Bean Scripting Framework

The Bean Scripting Framework is an attempt to create an API to allow calling scripting languages from Java. It hasn’t been updated for long and abandoned in favor of the standard JSR-223 API.
The BSF engine for Groovy is implemented by the org.codehaus.groovy.bsf.GroovyEngine class. However, that fact is normally hidden away by the BSF APIs. You just treat Groovy like any of the other scripting languages via the BSF API.

Since Groovy has its own native support for integration with Java, you only need to worry about BSF if you also want to also be able to call other languages, e.g. JRuby or if you want to remain very loosely coupled from your scripting language.
2.1. Getting started

Provided you have Groovy and BSF jars in your classpath, you can use the following Java code to run a sample Groovy script:

String myScript = "println('Hello World')\n  return [1, 2, 3]";
BSFManager manager = new BSFManager();
List answer = (List) manager.eval("groovy", "myScript.groovy", 0, 0, myScript);
assertEquals(3, answer.size());
2.2. Passing in variables

BSF lets you pass beans between Java and your scripting language. You can register/unregister beans which makes them known to BSF. You can then use BSF methods to lookup beans as required. Alternatively, you can declare/undeclare beans. This will register them but also make them available for use directly in your scripting language. This second approach is the normal approach used with Groovy. Here is an example:

BSFManager manager = new BSFManager();
manager.declareBean("xyz", 4, Integer.class);
Object answer = manager.eval("groovy", "test.groovy", 0, 0, "xyz + 1");
assertEquals(5, answer);
2.3. Other calling options

The previous examples used the eval method. BSF makes multiple methods available for your use (see the BSF documentation for more details). One of the other available methods is apply. It allows you to define an anonymous function in your scripting language and apply that function to arguments. Groovy supports this function using closures. Here is an example:

BSFManager manager = new BSFManager();
Vector<String> ignoreParamNames = null;
Vector<Integer> args = new Vector<Integer>();
args.add(2);
args.add(5);
args.add(1);
Integer actual = (Integer) manager.apply("groovy", "applyTest", 0, 0,
        "def summer = { a, b, c -> a * 100 + b * 10 + c }", ignoreParamNames, args);
assertEquals(251, actual.intValue());
2.4. Access to the scripting engine

Although you don’t normally need it, BSF does provide a hook that lets you get directly to the scripting engine. One of the functions which the engine can perform is to invoke a single method call on an object. Here is an example:

BSFManager manager = new BSFManager();
BSFEngine bsfEngine = manager.loadScriptingEngine("groovy");
manager.declareBean("myvar", "hello", String.class);
Object myvar = manager.lookupBean("myvar");
String result = (String) bsfEngine.call(myvar, "reverse", new Object[0]);
assertEquals("olleh", result);
3. JSR 223 javax.script API

JSR-223 is a standard API for calling scripting frameworks in Java. It is available since Java 6 and aims at providing a common framework for calling multiple languages from Java. Groovy provides its own richer integration mechanisms, and if you don’t plan to use multiple languages in the same application, it is recommended that you use the Groovy integration mechanisms instead of the limited JSR-223 API.
Here is how you need to initialize the JSR-223 engine to talk to Groovy from Java:

import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
...
ScriptEngineManager factory = new ScriptEngineManager();
ScriptEngine engine = factory.getEngineByName("groovy");
Then you can execute Groovy scripts easily:

Integer sum = (Integer) engine.eval("(1..10).sum()");
assertEquals(new Integer(55), sum);
It is also possible to share variables:

engine.put("first", "HELLO");
engine.put("second", "world");
String result = (String) engine.eval("first.toLowerCase() + ' ' + second.toUpperCase()");
assertEquals("hello WORLD", result);
This next example illustrates calling an invokable function:

import javax.script.Invocable;
...
ScriptEngineManager factory = new ScriptEngineManager();
ScriptEngine engine = factory.getEngineByName("groovy");
String fact = "def factorial(n) { n == 1 ? 1 : n * factorial(n - 1) }";
engine.eval(fact);
Invocable inv = (Invocable) engine;
Object[] params = {5};
Object result = inv.invokeFunction("factorial", params);
assertEquals(new Integer(120), result);
The engine keeps per default hard references to the script functions. To change this you should set a engine level scoped attribute to the script context of the name #jsr223.groovy.engine.keep.globals with a String being phantom to use phantom references, weak to use weak references or soft to use soft references - casing is ignored. Any other string will cause the use of hard references.