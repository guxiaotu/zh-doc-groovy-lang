编码风格
=========

Java 开发人员在开始学习 ``Groovy`` 时通常都会保持着 Java 的思想，但随着逐步的学习 ``Groovy`` ，一步一个脚印，会更加熟练编写 ``Groovy``。
本章节主要为这样的读者服务，介绍 ``Groovy`` 的一些通用的句法，操作以及一些新的特性，例如：闭包。

无需分号
--------

在 ``C / C++ / C# / Java`` 中无处不在的使用的着分号。
``Groovy`` 可以支持 99% 的 ``Java`` 语法，甚至可以直接将 ``Java`` 代码复制到 ``Groovy`` 中使用，这样你会看到到处都是分号。在 ``Groovy`` 中分号不是必须，你可以
忽略它们，当然最好是直接删除掉。


``return`` 为可选

在 ``Groovy`` 可以无需 ``return`` 关键字，方法中最后一个表达式将作为返回值。
在简短的方法或闭包中，这种方式非常的友好：

.. code-block:: groovy

	String toString() { return "a server" }
	String toString() { "a server" }

当使用变量作为返回时，有时的体验就并不是很好：

.. code-block:: groovy

	def props() {
	    def m1 = [a: 1, b: 2]
	    m2 = m1.findAll { k, v -> v % 2 == 0 }
	    m2.c = 3
	    m2
	}

在这种情况下，在最后一个表达式之前添加一行，或明确的使用 ``return`` 将会有更好的可读性。	
对于我自己而言有时使用 ``return`` 有时不使用，这完全属于自己的喜好。但是在闭包中，我们更多情况下是不使用 ``return`` 。
尽管 ``return`` 作为关键字是可选的，但也并不意味着强制不可使用，这还是完全取决于你代码的可读性。

这里需要注意的是，当使用 ``def`` 来定义方法的具体类型，方法的返回取决于其最后的表达式的返回值。
通常更倾向于使用具体的返回类型，如 ： ``void`` 或 类型。
在上面例子中，想象如果我们忘记写最后一行 ``m2``，这里最后一行的表达式将是 ``m2.c = 3`` , 其放回值就会是 3，而不是你所期望的 ``map``. 



Statements like if/else, try/catch can thus return a value as well, as there’s a "last expression" evaluated in those statements:

.. code-block:: groovy

	def foo(n) {
	    if(n == 1) {
	        "Roshan"
	    } else {
	        "Dawrani"
	    }
	}

	assert foo(1) == "Roshan"
	assert foo(2) == "Dawrani"

Def and type
------------

当我们谈论到 ``def`` 和 ``types``, 我们会看到开发人员同时使用 ``def`` 和响应的类型。但是在这里 ``def`` 是多余的。你可以选择是使用 ``def`` 或 具体类型。

不要这样写：

.. code-block:: groovy

	def String name = "Guillaume"

可以写成：

.. code-block:: groovy

	String name = "Guillaume"

当使用 ``def`` 时，其实际对应 ``Object`` 类型（这样你可以使用 ``def`` 定义任何变量，如果方法声明返回 ``def`` ，其可以返回任意类型对象）。	

对于方法中无类型的参数，我们可以使用 ``def`` 定义，虽然这也不是必须的，并且我们更倾向于忽略类型：

例如：

.. code-block:: groovy

	void doSomething(def param1, def param2) { }

我们更喜欢写成:

.. code-block:: groovy

	void doSomething(param1, param2) { }

上一章节我们提到过，明确方法参数的类型是比较好的，可以帮助完善代码的记录， ``IDEs`` 的代码补全，以及有利用 ``Groovy`` 中静态类型检查及静态编译的能力。	

当在定义构造方法时， ``def`` 是应该避免使用的：

.. code-block:: groovy

	class MyClass {
	    def MyClass() {}
	}

应当删除 ``def``:

.. code-block:: groovy

	class MyClass {
	    MyClass() {}
	}

默认为 Public （Public by default）
------------------------------------

``Groovy`` 中 ``classes`` 和方法都默认为 ``public`` . 
你可以不必使用 ``public`` 修饰符，如果其不为 ``public`` 可以使用其他修饰符:

例如：

.. code-block:: groovy

	public class Server {
	    public String toString() { return "a server" }
	}

可以替换为:

.. code-block:: groovy

	class Server {
	    String toString() { "a server" }
	}

你可能希望可见范围在 ``package-scope`` ，我们可以使用注解来满足所期望的可见性：	
 
.. code-block:: groovy

	class Server {
	    @PackageScope Cluster cluster
	}


省略括号
----------

``Groovy`` 允许在顶级表达式中省略括号：

.. code-block:: groovy

	println "Hello"
	method a, b
	//	vs:

	println("Hello")
	method(a, b)

当使用闭包作为方法的最后一个参数时，就像 ``each{}`` 迭代调用，你可以将闭包放置在右括号之外，甚至省略括号：	

.. code-block:: groovy

	list.each( { println it } )
	list.each(){ println it }
	list.each  { println it }

第三种形式表达更自然，空括号带来无用的句法噪音。

在一些情况下是不可以参数括号。像前面提到的，在顶层的表达式中可以省略括号，但是在嵌套方法调用，赋值的右侧，其括号是不可以省略的：

.. code-block::  groovy

	def foo(n) { n }

	println foo 1 // won't work
	def m = foo 1


Classes as first-class citizens
-------------------------------

``.classes`` 的后缀在 ``Groovy`` 中并非必须，有点像 ``Java`` 中的 instanceof。

例如：

.. code-block:: groovy

	connection.doPost(BASE_URI + "/modify.hqu", params, ResourcesResponse.class)

Using GStrings we’re going to cover below, and using first class citizens:

.. code-block:: groovy

	connection.doPost("${BASE_URI}/modify.hqu", params, ResourcesResponse)


Getters and Setters
-------------------

在 ``Groovy`` 中使用 ``getters`` 和 ``setters`` 来调用属性，并提供简单符号访问及修改属性。
替代 Java 中使用 ``getters / setters`` 的方式，你可以直接访问属性:

.. code-block:: groovy

	resourceGroup.getResourcePrototype().getName() == SERVER_TYPE_NAME
	resourceGroup.resourcePrototype.name == SERVER_TYPE_NAME

	resourcePrototype.setName("something")
	resourcePrototype.name = "something"

在 ``Groovy`` 中编写 beans，其被称为 ``POGOs （Plain Old Groovy Objects）`` , 你可以无需自己创建属性的  ``getter / setter`` , ``Groovy`` 编译器会为
你完成这些:	

.. code-block:: groovy


	class Person {
	    private String name
	    String getName() { return name }
	    void setName(String name) { this.name = name }
	}

你可以这么写：

.. code-block:: groovy

	class Person {
	    String name
	}

就像你看到的，一个不带访问修饰符的 ``field`` ，``Groovy`` 编译器会为其生成 private field 及 getter 和 setter 方法。

When using such POGOs from Java, the getter and setter are indeed there, and can be used as usual, of course.

尽管编译可以创建 ``getter/setter`` 逻辑，如果你需要扩展这些 ``getter/setter`` ， 你可以重写这些方法，其编译器将会使用你的重构逻辑替代默认的。


使用命名参数初始化 beans
--------------------------------------------------------------------


With a bean like:

.. code-block:: groovy

	class Server {
	    String name
	    Cluster cluster
	}


替代调用 ``setter`` 方法，我们可以像下面这样：

.. code-block:: groovy

	def server = new Server()
	server.name = "Obelix"
	server.cluster = aCluster

你可以使用默认的构造函数为参数赋值（首先调用构造方法，通过指定的 ``map`` 内容，依次调用对应的 ``setter`` 方法）

.. code-block:: groovy

	def server = new Server(name: "Obelix", cluster: aCluster)

Using with() for repeated operations on the same bean
---------------------------------------------------------

当创建实例时，使用构造函数对变量进行赋值是很容易，但是当你需要更新实例中的变量，你会不断的重复使用 ``server`` 前缀，对吗？
不过现在你可以感谢 ``with()`` 方法，它将省去前缀的重复，它被引入到 ``Groovy`` 中所的对象中：

.. code-block:: groovy

	server.name = application.name
	server.status = status
	server.sessionCount = 3
	server.start()
	server.stop()

vs:

.. code-block:: groovy

	server.with {
	    name = application.name
	    status = status
	    sessionCount = 3
	    start()
	    stop()
	}


Equals and == 
--------------

Java’s ``==`` 就是 Groovy’s ``is()`` ， Groovy’s ``==`` 较于 ``equals()`` 更强大。

比较对象的引用，你可以 ``a.is(b)`` 替代 ``==`` 。

通常使用 ``equals()`` 进行比较，在 ``Groovy`` 更多使用 ``==``，其可以很好的避免 ``NullPointerException``:

Instead of:

.. code-block:: groovy

	status != null && status.equals(ControlConstants.STATUS_COMPLETED)

Do:

.. code-block:: groovy

	status == ControlConstants.STATUS_COMPLETED

GStrings (插值，多行赋值)
-------------------------

在 Java 中我们使用字符串及多变量结合，会使用大量的双引号，加号以及 ``\n`` 字符用于换行。

比较以下方式：

.. code-block:: groovy

	throw new Exception("Unable to convert resource: " + resource)

vs:

.. code-block:: groovy

	throw new Exception("Unable to convert resource: ${resource}")

在花括号中，你可以添加任何表达式，而不仅仅是变量。
对于简单的变量或 ``variable.property`` ，你甚至可以不使用花括号：	

.. code-block:: groovy

	throw new Exception("Unable to convert resource: $resource")

你甚至可以延迟执行表达式，使用闭包的符号 ``${-> resource}`` .
当 ``GString`` 被转化为字符串时，开始执行闭包并返回 ``toString`` 。

Example:

.. code-block:: groovy

	int i = 3

	def s1 = "i's value is: ${i}"
	def s2 = "i's value is: ${-> i}"

	i++

	assert s1 == "i's value is: 3" // eagerly evaluated, takes the value on creation
	assert s2 == "i's value is: 4" // lazily evaluated, takes the new value into account

在 Java 中字符串及其连接表达式非常冗长：

.. code-block:: java

	throw new PluginException("Failed to execute command list-applications:" +
	    " The group with name " +
	    parameterMap.groupname[0] +
	    " is not compatible group of type " +
	    SERVER_TYPE_NAME)

You can use the \ continuation character (this is not a multiline string):
你可以使用 ``\`` 延续字符（这里并不是多行字符串）：

.. code-block:: groovy

	throw new PluginException("Failed to execute command list-applications: \
	The group with name ${parameterMap.groupname[0]} \
	is not compatible group of type ${SERVER_TYPE_NAME}")

或使用三引号来使用多行字符串：

.. code-block:: groovy

	throw new PluginException("""Failed to execute command list-applications:
	    The group with name ${parameterMap.groupname[0]}
	    is not compatible group of type ${SERVER_TYPE_NAME)}""")

你可以使用 ``.stripIndent()`` 删除多行字符串左侧缩进。

请注意 ``Groovy`` 中单引号与双引号的差别：
单引号创建 Java 字符串，不允许插入变量，双引号在插入变量时创建 ``GStrings`` ，否则创建 Java 字符串。

你可使用三引号来创建多行字符串：三双引号用于 ``GStrings`` ，三单引号用于 ``Strings``.

如果你需要使用正则表达式，你需要使用 ``slashy`` 符号：

.. code-block:: groovy

	assert "foooo/baaaaar" ==~ /fo+\/ba+r/

``slashy`` 的优势不使用双反斜杠，并使正则表达式使用更为简单。
（The advantage of the "slashy" notation is that you don’t need to double escape backslashes, making working with regex a bit simpler.）

通常使用单引号来创建字符串常量，使用双引号来处理需要插值字符串。

Native syntax for data structures
---------------------------------

``Groovy`` 提供原生句法构建数据结构，如：``lists``, ``maps``, ``regex``, ``ranges``。
请确保在编程过程中是这么使用的。

这里有一些例子：

.. code-block:: groovy

	def list = [1, 4, 6, 9]

	// by default, keys are Strings, no need to quote them
	// you can wrap keys with () like [(variableStateAcronym): stateName] to insert a variable or object as a key.
	def map = [CA: 'California', MI: 'Michigan']

	def range = 10..20
	def pattern = ~/fo*/

	// equivalent to add()
	list << 5

	// call contains()
	assert 4 in list
	assert 5 in list
	assert 15 in range

	// subscript notation
	assert list[1] == 4

	// add a new key value pair
	map << [WA: 'Washington']
	// subscript notation
	assert map['CA'] == 'California'
	// property notation
	assert map.WA == 'Washington'

	// matches() strings against patterns
	assert 'foo' =~ pattern

The Groovy Development Kit
-------------------------------

继续以上数据结构，当在集合上使用迭代器，``Groovy`` 提供了多种扩展方法，包装了 Java’s 核心数据结构， 例如 ： ``each()``, ``find()``, ``findAll()``, ``every()``, ``collect()``, ``inject()``. 
这些方法引入到编程语言中，是复杂的计算变得更为容易。
由于动态语言的特性，大量的新方法才能被运用到各种类型中。
你会发现很多有用的方法在 String, Files, Streams, Collections 等上：

	http://beta.groovy-lang.org/gdk.html

The Power of switch
-------------------

Groovy 中 ``switch`` 较之 ``C-ish`` 语言更为强大，后者只能处理原始类型。
Groovy 中可以处理的几乎所有类型：

.. code-block:: groovy

	def x = 1.23
	def result = ""
	switch (x) {
	    case "foo": result = "found foo"
	    // lets fall through
	    case "bar": result += "bar"
	    case [4, 5, 6, 'inList']:
	        result = "list"
	        break
	    case 12..30:
	        result = "range"
	        break
	    case Integer:
	        result = "integer"
	        break
	    case Number:
	        result = "number"
	        break
	    case { it > 3 }:
	        result = "number > 3"
	        break
	    default: result = "default"
	}
	assert result == "number"

一般，在类型上 ``isCase()`` 方法可以判断是否符合条件。	

Import aliasing
---------------

在 Java 中，当使用名字相同的两个类，可以通过识别其来自不同的包，例如： ``java.util.List`` 和 ``java.awt.List``，你可以引入一个类，另一个则需要全路径名称。

在一些情况下，你的代码中使用很长的类名称，会使得代码变得很累赘。

为了改进这种情况，``Groovy`` 中引入了别名：

.. code-block:: groovy

	import java.util.List as juList
	import java.awt.List as aList

	import java.awt.WindowConstants as WC

也可以引入静态方法：	

.. code-block:: groovy

	import static pkg.SomeClass.foo
	foo()

Groovy Truth
------------

所有对象都可以转化为布尔值：任何 ``null``, ``void`` , 等于零或为空将返回 ``false``, 否则返回 ``true``.

So instead of writing:

.. code-block:: groovy

	if (name != null && name.length > 0) {}

You can just do:

.. code-block:: groovy

	if (name) {}

同样适用于集合。	

Thus, you can use some shortcuts in things like while(), if(), the ternary operator, the Elvis operator (see below), etc.

It’s even possible to customize the Groovy Truth, by adding an boolean asBoolean() method to your classes!

Safe graph navigation
---------------------

``Groovy`` 支持一种进化的 ``.`` 操作符，用于安全的获取对象的画像。

在 ``Java`` 中当我们获取对象的节点时，需要去检查其是否为 ``null``，你通常需要编写一些复杂的 ``if`` 语句，如：

.. code-block:: Java

	if (order != null) {
	    if (order.getCustomer() != null) {
	        if (order.getCustomer().getAddress() != null) {
	            System.out.println(order.getCustomer().getAddress());
	        }
	    }
	}

使用 ``?.`` 操作符，你可以将代码简化成这样：	

.. code-block:: groovy

	println order?.customer?.address

``Nulls`` 通过调用链检查，当其中任何元素为 ``null``，不会抛出 ``NullPointerException`` ， 而会返回 ``null``.	

Assert
---------

可以使用断言声明检查参数，返回值等等。

Contrary to Java’s assert, ``assert`s`` don’t need to be activated to be working, so ``assert`s`` are always checked.

.. code-block:: groovy

	def check(String name) {
	    // name non-null and non-empty according to Groovy Truth
	    assert name
	    // safe navigation + Groovy Truth to check
	    assert name?.size() > 3
	}

你将会注意到 ``Groovy`` 的  ``Power Assert`` 输出内容相当丰富，其描述	每个子表达式的断言视图。

Elvis operator for default values
---------------------------------

``Elvis operator`` 是一个比较特殊的三元操作符，其使用非常便利：

We often have to write code like:

.. code-block:: groovy

	def result = name != null ? name : "Unknown"

Thanks to Groovy Truth, the null check can be simplified to just 'name'.

可以这样使用 ``Elvis operator`` :

.. code-block:: groovy

	def result = name ?: "Unknown"

Catch any exception
-------------------

如果你并不关心你 try block 中的异常类型，你可以简单的捕获他们，并忽略其捕获的异常类型：
If you don’t really care about the type of the exception which is thrown inside your try block, you can simply catch any of them and simply omit the type of the caught exception. So instead of catching the exceptions like in:

.. code-block:: groovy

	try {
	    // ...
	} catch (Exception t) {
	    // something bad happens
	}

Then catch anything ('any' or 'all', or whatever makes you think it’s anything):

.. code-block:: groovy

	try {
	    // ...
	} catch (any) {
	    // something bad happens
	}

请注意这里捕获的是 ``Exceptions``  而不是 ``Throwable‘s``。
如果你需要捕获 ``everything`` , 你需要明确你所期望捕获的 ``Throwable's``.

Optional typing advice
----------------------


I’ll finish on some words on when and how to use optional typing. Groovy lets you decide whether you use explicit strong typing, or when you use def.

I’ve got a rather simple rule of thumb: whenever the code you’re writing is going to be used by others as a public API, you should always favor the use of strong typing, it helps making the contract stronger, avoids possible passed arguments type mistakes, gives better documentation, and also helps the IDE with code completion. Whenever the code is for your use only, like private methods, or when the IDE can easily infer the type, then you’re more free to decide when to type or not.

