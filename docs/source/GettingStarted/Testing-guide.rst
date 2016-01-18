Test Guide
==========

介绍
------

``Groovy`` 中对于编写测试有良好的支持。除了语言特性及集成测试上拥有先进的库及框架，``Groovy`` 生态系统中也诞生了大量的测试库及框架。

本章节开始介绍测试特性，并继续关注 ``JUnit`` 集成，``Spock`` 规范以及 ``Geb`` 用于功能性测试。
最后，我们也会介绍其他一些测试库在 ``Groovy`` 中的使用。


语言特性
---------

除了 ``JUnit`` 集成支持， ``Groovy`` 语言特性已经被证明在测试驱动开发中非常有价值。
这一章节讲展开讲解。

强大的断言
^^^^^^^^^^^^^^

编写测试用例也意味着使用断言制定一系列假设。``Java`` 中可以使用 ``assert`` 关键字（``J2SE 1.4`` 中添加）。
``Java`` 中断言的使用可以通过设置 ``JVM`` 参数 ``-ea (or -enableassertions)`` 和 ``-da (or -disableassertions)`` 生效与失效。默认情况下是不生效。

``Groovy`` 中自带了功能相当强大的一些断言声明。与 ``Java`` 中不同的是，在断言返回结果为 ``false`` 的情况的输出：

.. code-block:: groovy

	def x = 1
	assert x == 2

	// Output:  					// <1>           
	//
	// Assertion failed:
	// assert x == 2
	//        | |
	//        1 false

<1> 这一部分内容将在 ``std-err`` 中输出

当断言为不成功时，将抛出 ``java.lang.AssertionError``，其中包括了一些原始异常信息。断言的输出会展现出其内部表现情况。

下面例子中，我们会看见强大的断言声明在复杂的布尔语句，集合以及 ``toString`` 中发挥的强大作用：

.. code-block:: groovy

	def x = [1,2,3,4,5]
	assert (x << 6) == [6,7,8,9,10]

	// Output:
	//
	// Assertion failed:
	// assert (x << 6) == [6,7,8,9,10]
	//         | |     |
	//         | |     false
	//         | [1, 2, 3, 4, 5, 6]
	//         [1, 2, 3, 4, 5, 6]

和 ``Java`` 中另一个重要的不同是，``Groovy`` 中的断言是默认有效的。这是一项在语言设计层面的决定。

.. epigraph::

   it makes no sense to take off your swim ring if you put your feet into real water

   -- Bertrand Meyer


断言中需要注意布尔表达式中带有副作用的方法。内部的错误信息构建机制，并不存储当前目标实例的引用，这样错误信息在输出时产生一定失真：

.. code-block:: groovy

	assert [[1,2,3,3,3,3,4]].first().unique() == [1,2,3]

	// Output:
	//
	// Assertion failed:
	// assert [[1,2,3,3,3,3,4]].first().unique() == [1,2,3]
	//                          |       |        |
	//                          |       |        false
	//                          |       [1, 2, 3, 4]
	//                          [1, 2, 3, 4]     							// <1>      

<1> 错误信息显示的是集合的状态，而不是在 ``unique`` 方法执行前的状态。

你也可以使用 Java 的语法 ``assert expression1 : expression2``（``expression1`` 中判断条件，``expression2`` 为自定义输出结果 ） 来自定义断言错误信息的输出。
要知道，这样将会使断言失去其强大的功能，完全就依赖自定义错误信息。

Mocking and Stubbing
^^^^^^^^^^^^^^^^^^^^
Groovy 内部可以很好的支持 ``Mocking and Stubbing`` 。``Java`` 中，动态 ``mocking`` 框架非常流行。
这个关键原因使用 ``Java`` 手工定制来创建 ``mock`` 是一项十分困难的工作。这些框架在 ``Groovy`` 中可以很方便的使用，如果你选择自定义一些 ``mock`` 在 ``Groovy`` 中也是很简单的。

下面章节将介绍使用 ``Groovy`` 语言特性来创建 ``mocks`` 和 ``stubs``。


Map Coercion
""""""""""""

通过使用 maps 或 expandos ，我们可以很容易获取期望行为结果：

.. code-block:: groovy

	class TranslationService {
	    String convert(String key) {
	        return "test"
	    }
	}

	def service = [convert: { String key -> 'some text' }] as TranslationService
	assert 'some text' == service.convert('key.text')

操作符 ``as`` 用于强制转换 map 为特定的类型。 map 的 keys 对应方法的名称，其 values 为 ``groovy.lang.Closure`` 闭包块，被指向对应的方法块。


Be aware that map coercion can get into the way if you deal with custom java.util.Map descendant classes in combination with the as operator. The map coercion mechanism is targeted directly at certain collection classes, it doesn’t take custom classes into account.

Closure Coercion
""""""""""""""""
在 ``closures`` 上使用 ``as`` 这种方式非常适合开发者测试一些简单的场景。这项技术还不至于让我们放弃使用动态 ``mocking``。

``Classes`` 或 ``interfaces`` 中只有一个方法，可以使用强制转化为闭包块为给定的类型。需要注意，在这个过程中，Groovy 内部将创建一个继承给定类型的代理对象。

看看下面例子中，转换闭包为指定类型：

.. code-block:: groovy

	def service = { String key -> 'some text' } as TranslationService
	assert 'some text' == service.convert('key.text')

``Groovy`` 支持称为 ``SAM 强制转换`` 的特性。在这种情况下，可以不使用 ``as`` 操作符， 运行环境可以推断其目标 ``SAM`` 类型。  

.. code-block:: groovy

	abstract class BaseService {
	    abstract void doSomething()
	}

	BaseService service = { -> println 'doing something' }
	service.doSomething()


MockFor and StubFor
"""""""""""""""""""

``Groovy`` 中的 ``mocking`` 和 ``stubbing`` 类可以在 ``groovy.mock.interceptor`` 包中找到。

``MockFor`` 通过定义强一致顺序期望行为，来支持单元测试的隔离性。在一个典型的测试场景涉及一个测试类，以及一个或多个协作的支持类。在这个场景中，通常只希望对业务逻辑类进行测试。针对这种情况的一个策略就是将当前协作类替换为简单的 ``mock`` 对象，以便隔离出业务逻辑来进行测试。``MockFor`` 这里创建的 ``mocks`` 使用元编程。这些协助类行为被定义为行为规范，并且自动监测，强制执行。

.. code-block:: groovy

	class Person {
	    String first, last
	}

	class Family {
	    Person father, mother
	    def nameOfMother() { "$mother.first $mother.last" }
	}
	def mock = new MockFor(Person)      						// <1>
	mock.demand.getFirst{ 'dummy' }
	mock.demand.getLast{ 'name' }
	mock.use {                          						// <2>
	    def mary = new Person(first:'Mary', last:'Smith')
	    def f = new Family(mother:mary)
	    assert f.nameOfMother() == 'dummy name'
	}
	mock.expect.verify()                						// <3>

<1> a new mock is created by a new instance of MockFor

<2> a Closure is passed to use which enables the mocking functionality

<3> a call to verify checks whether the sequence and number of method calls is as expected

``StubFor`` 通过定义松散顺序期望行为，来支持单元测试的隔离性。在一个典型的测试场景涉及一个测试类，以及一个或多个协作的支持类。在这个场景中，通常只希望测试 CUT 的业务逻辑。
针对这种情况的一个策略就是将当前协作类替换为简单的 ``stub`` 对象，以便隔离出业务逻辑来进行测试。``StubFor`` 这里创建的 ``stubs`` 使用元编程。这些协助类行为被定义为行为规范。

.. code-block:: groovy

	def stub = new StubFor(Person)      						// <1>
	stub.demand.with {                  						// <2>
	    getLast{ 'name' }
	    getFirst{ 'dummy' }
	}
	stub.use {                          						// <3>
	    def john = new Person(first:'John', last:'Smith')
	    def f = new Family(father:john)
	    assert f.father.first == 'dummy'
	    assert f.father.last == 'name'
	}
	stub.expect.verify()                						// <4>


<1> a new stub is created by a new instance of StubFor
<2> the with method is used for delegating all calls inside the closure to the StubFor instance
<3> a Closure is passed to use which enables the stubbing functionality
<4> a call to verify (optional) checks whether the number of method calls is as expected
MockFor and StubFor can not be used to test statically compiled classes e.g for Java classes or Groovy classes that make use of @CompileStatic. To stub and/or mock these classes you can use Spock or one of the Java mocking libraries.

Expando Meta-Class (EMC)
""""""""""""""""""""""""""

``Groovy`` 中有个特殊的 ``MetaClass`` 被称为 ``ExpandoMetaClass``。
其可以使用简洁的闭包语法动态创建方法，构造器，属性以及静态方法。

每个 ``java.lang.Class`` 都有一个特别的 ``metaClass`` 属性，其指向一个 ``ExpandoMetaClass`` 实例。
看看下面例子：

.. code-block:: groovy

	String.metaClass.swapCase = {->
	    def sb = new StringBuffer()
	    delegate.each {
	        sb << (Character.isUpperCase(it as char) ? Character.toLowerCase(it as char) :
	            Character.toUpperCase(it as char))
	    }
	    sb.toString()
	}

	def s = "heLLo, worLD!"
	assert s.swapCase() == 'HEllO, WORld!'

``ExpandoMetaClass`` 是一个非常好的  ``mocking`` 功能备选方案，并且其可以提供更多的高级功能例如 ``mocking`` 静态方法。

.. code-block:: groovy

	class Book {
	    String title
	}

	Book.metaClass.static.create << { String title -> new Book(title:title) }

	def b = Book.create("The Stand")
	assert b.title == 'The Stand'

也可以 ``moking`` 构造函数。

.. code-block:: groovy

	Book.metaClass.constructor << { String title -> new Book(title:title) }

	def b = new Book("The Stand")
	assert b.title == 'The Stand'

``Mocking`` 构造函数像是 ``hack`` ，这种方式对一些用例是相当有作用的。在 ``Grails`` 中可以找到例子，使用 ``ExpandoMetaClass`` 可以在运行时，对领域类添加构造方法。这样可以将 领域对象自身注册到
``Spring Application context`` ， 并且允许注入 ``services`` 或 被依赖注入容器管理的其他 ``beans``。

如果你希望在测试方法级别，改变 ``metaClass`` 属性，你需要删除这些变化，否则这些变化将会持久化。删除变化需要这样做：

.. code-block:: groovy

	GroovySystem.metaClassRegistry.setMetaClass(java.lang.String, null)

另一种替代方式，使用 ``MetaClassRegistryChangeEventListener`` ，跟踪 ``classes`` 变化，在测试运行环境中
通过 cleanup 方法删除这些变化。``Grails`` 框架中有非常好的例子。

除了在类这一级别使用 ``ExpandoMetaClass`` ，在 ``per-object`` 这一级别也可以支持使用：

.. code-block:: groovy

	def b = new Book(title: "The Stand")
	b.metaClass.getTitle {-> 'My Title' }

	assert b.title == 'My Title'

在这里 ``meta-class`` 仅与实例关联。取决于应用的场景，在这里这种方式较于全局的 ``meta-class`` 设置更为合适。


GDK Methods
^^^^^^^^^^^

接下来的章节将详细介绍 ``GDK`` ，其可以在测试用例中使用的方式，下面的例子介绍测试数据的生成。

Iterable#combinations
"""""""""""""""""""""

``combinations`` 方法被加入到 ``java.lang.Iterable`` 类中，用于获取从多个 list 中的组合 list:

.. code-block:: groovy

	void testCombinations() {
	    def combinations = [[2, 3],[4, 5, 6]].combinations()
	    assert combinations == [[2, 4], [3, 4], [2, 5], [3, 5], [2, 6], [3, 6]]
	}

这个方法可以用于在指定的方法调用上生成所有可能的参数组合。

Iterable#eachCombination
""""""""""""""""""""""""
``eachCombination`` 方法被加入到 ``java.lang.Iterable``，当组合建立时，可以在每个元素上运行相应功能：
``eachCombination`` 是 ``GDK`` 中方法，满足 ``java.lang.Iterable`` 接口的所有类都包含此方法。
在 lists 每次组合中均会调用：

.. code-block:: groovy

	void testEachCombination() {
	    [[2, 3],[4, 5, 6]].eachCombination { println it[0] + it[1] }
	}

The method could be used in the testing context to call methods with each of the generated combinations.

Tool Support
^^^^^^^^^^^^

Test code coverage
""""""""""""""""""

代码覆盖对于测量单元测试的效果是很有用的。高代码覆盖的程序出现严重 bug 的机率要远低于低覆盖甚至无覆盖的程序。
为获取代码覆盖指标，需要在测试之前，对生成的字节码进行测量。``Groovy`` 中提供了 `Cobertura <http://cobertura.github.io/cobertura/>`_ 工具用于检测此指标数据。

各种框架及构建工具也集成 ``Cobertura``. ``Grails`` 中代码覆盖插件基于 ``Cobertura`` ;  ``Gradle`` 中使用 ``gradle-cobertura`` 插件，看名字就知道其来源。

下面例子中将展示 ``Cobertura`` 是如何在 ``Gradle`` 构建脚本中能够测试覆盖：

.. code-block:: groovy

	def pluginVersion = '<plugin version>'
	def groovyVersion = '<groovy version>'
	def junitVersion = '<junit version>'

	buildscript {
	    repositories {
	        mavenCentral()
	    }
	    dependencies {
	        classpath 'com.eriwen:gradle-cobertura-plugin:${pluginVersion}'
	    }
	}

	apply plugin: 'groovy'
	apply plugin: 'cobertura'

	repositories {
	    mavenCentral()
	}

	dependencies {
	    compile "org.codehaus.groovy:groovy-all:${groovyVersion}"
	    testCompile "junit:junit:${junitVersion}"
	}

	cobertura {
	    format = 'html'
	    includes = ['**/*.java', '**/*.groovy']
	    excludes = ['com/thirdparty/**/*.*']
	}

``Cobertura`` 覆盖报告有多重输出格式选择，可以代码覆盖报告可以集成到构建任务中。

Unit Tests with JUnit 3 and 4
-----------------------------

``Groovy`` 简化了 ``JUnit`` 的测试， 使其更 ``Groovy``.
下面章节将更加详细介绍 JUnit3/4 与 Groovy 的集成。

JUnit3
^^^^^^

在 Groovy 中支持 JUnit3 测试最突出的类就是 ``GroovyTestCase``。其源自 ``junit.framework.TestCase`` ，并提供了一系列附加方法使得 Groovy 中的测试更为便利。

尽管 ``GroovyTestCase`` 继承自 ``TestCase`` , 但并不意味着无法使用 Junit4 中的特性。
在最近的 ``Groovy`` 版本中捆绑了 Junit4，并且使 ``TestCase`` 可以向后兼容。
在 Groovy mailing-list 中也有关于使用 Junit4 还是 GroovyTestCase 的争论，这个争论结果主要还是一个喜好问题。
但是在使用 ``GroovyTestCase`` 上可以灵活的使用一些方法，使用测试更容易编写

在这一章节，我们将会看到 ``GroovyTestCase`` 提供的一些方法。
完整的方法列表可以在 JavaDoc 中 `groovy.util.GroovyTestCase <http://docs.groovy-lang.org/2.4.5/html/gapi/index.html?groovy/util/GroovyTestCase.html>`_ 找到。
不要忘记它是继承于 ``junit.framework.TestCase`` ，继承了所有 ``assert*`` 方法。

Assertion Methods
"""""""""""""""""

``GroovyTestCase`` 继承于 ``junit.framework.TestCase`` ，继承了所有 ``assert*`` 方法

.. code-block:: groovy

	class MyTestCase extends GroovyTestCase {

	    void testAssertions() {
	        assertTrue(1 == 1)
	        assertEquals("test", "test")

	        def x = "42"
	        assertNotNull "x must not be null", x
	        assertNull null

	        assertSame x, x
	    }

	}

与 Java 相比，这里代码在大多数情况下，省略了括号，这样使得 Junit 断言方法调用表达式的可读性更强。

``GroovyTestCase`` 其中一个有趣的断言方法就是 ``assertScript``。它将确保给出的 groovy 代码中没有任何异常：

.. code-block:: groovy

	void testScriptAssertions() {
	    assertScript '''
	        def x = 1
	        def y = 2

	        assert x + y == 3
	    '''
	}

shouldFail Methods
""""""""""""""""""

``shouldFail`` 被用于检查代码块是否有误。如果有误，断言会将其吞掉，否则抛出异常：

.. code-block:: groovy

	void testInvalidIndexAccess1() {
	    def numbers = [1,2,3,4]
	    shouldFail {
	        numbers.get(4)
	    }
	}

上面代码使用了 ``shouldFail`` 基础的方法接口，使用 ``groovy.lang.Closure`` 作为唯一参数。

如果我们想在 ``shouldFail`` 上使用指定的 ``java.lang.Exception`` 类型，可以这样做：

.. code-block::  groovy

	void testInvalidIndexAccess2() {
	    def numbers = [1,2,3,4]
	    shouldFail IndexOutOfBoundsException, {
	        numbers.get(4)
	    }
	}

到现在 ``shouldFail`` 还有个特性没有演示：返回异常信息。
如果你想断言异常的错误信息，这样将会非常有用：

.. code-block:: groovy

	void testInvalidIndexAccess3() {
	    def numbers = [1,2,3,4]
	    def msg = shouldFail IndexOutOfBoundsException, {
	        numbers.get(4)
	    }
	    assert msg.contains('Index: 4, Size: 4')
	}

notYetImplemented Method
""""""""""""""""""""""""

``notYetImplemented`` 方法受到 ``HtmlUnit`` 的极大影响。
其允许标记测试方法为未实现。标记 ``notYetImplemented`` 的方法在执行中失败，测试也会提示通过

.. code-block:: groovy

	void testNotYetImplemented1() {
	    if (notYetImplemented()) return   

	    assert 1 == 2                     
	}


这里可以使用注解 ``@NotYetImplemented`` 来替代 ``notYetImplemented`` 方法调用。
可以在方法上注释其为未实现，可以不需要再调用 ``notYetImplemented`` 方法：

.. code-block:: groovy

	@NotYetImplemented
	void testNotYetImplemented2() {
	    assert 1 == 2
	}

Junit4
^^^^^^
在 Groovy 中编写 Junit4 的测试用例没有任何限制。
在 Junit4 中可以使用 ``groovy.test.GroovyAssert`` 中提供的静态方法来替换 ``GroovyTestCase`` 中的方法：


.. code-block:: groovy

	import org.junit.Test

	import static groovy.test.GroovyAssert.shouldFail

	class JUnit4ExampleTests {

	    @Test
	    void indexOutOfBoundsAccess() {
	        def numbers = [1,2,3,4]
	        shouldFail {
	            numbers.get(4)
	        }
	    }

	}

在上面例子中看到，``GroovyAssert`` 中的静态方法被引入，可以想在 ``GroovyTestCase`` 中一样使用。

``groovy.test.GroovyAssert`` 继承 ``org.junit.Assert`` 这意味着，其继承了 ``Junit`` 中所有的断言方法。
然而通过介绍了强大的断言声明，使用断言声明来替代 Junit 断言方法是更好的做法，其主要的理由就是这样可以增强断言的消息内容。

这里需要注意的是 ``GroovyAssert.shouldFail`` 与 ``GroovyTestCase.shouldFail`` 是完全不同的。
``GroovyTestCase.shouldFail`` 返回异常消息， ``GroovyAssert.shouldFail`` 返回异常本身。这里可以访问异常对象中其他的一些属性及方法：

.. code-block:: groovy

	@Test
	void shouldFailReturn() {
	    def e = shouldFail {
	        throw new RuntimeException('foo',
	                                   new RuntimeException('bar'))
	    }
	    assert e instanceof RuntimeException
	    assert e.message == 'foo'
	    assert e.cause.message == 'bar'
	}

Testing with Spock
------------------

``Spock`` 是针对 ``Java`` 和 ``Groovy`` 应用的测试及规范框架。使其脱引而出的是其优美和极富表现力的规范 DSL
。
``Spock`` 规范使用 ``Groovy`` 类编写。尽管使用 ``Groovy`` 编写，他们也可用于测试 ``Java`` 类。

``Spock`` 可以用于单元，集成以及 ``BBD（behavior-driven-development）`` 测试，其自身并非测试框架或测试库。

除了这些特性，``Spock`` 也是一个很好的范例，用于实践如何利用 ``Groovy`` 编程语言的特性。

这一章节不会详细介绍如何使用 ``Spock`` ，而是会让大家记住它，是如何进行单元，继承，功能以及其他类型的测试。

接下来看一下 ``Spock`` 规范的结构。

It should give a pretty good feeling on what Spock is up to.

Specifications
^^^^^^^^^^^^^^

``Spock`` 让你编写规范来描述特性（性能，问题）来展现系统关注点。这里的 ``系统`` 可以是一个类或是一个完整的应用，用更规范的术语表达为基于规范的系统。

The feature description starts from a specific snapshot of the system and its collaborators, this snapshot is called the feature’s fixture.

``Spock`` 规范类源自 ``spock.lang.Specification``。
具体规范类由 fields, fixture methods, features methods 和 helper methods 组成。

下面看一个简单的规范对于单一特性方法：

.. code-block:: groovy

	class StackSpec extends Specification {

	    def "adding an element leads to size increase"() {              // <1>
	        setup: "a new stack instance is created"        			// <2>
	            def stack = new Stack()

	        when:                                           
	            stack.push 42											// <3>

	        then:                                           			// <4>
	            stack.size() == 1
	    }
	}

<1> Feature method, is by convention named with a String literal.

<2> Setup block, here is where any setup work for this feature needs to be done.

<3> When block describes a stimulus, a certain action under target by this feature specification.

<4> Then block any expressions that can be used to validate the result of the code that was triggered by the when block.

``Spock`` 特性规范被定义在 ``spock.lang.Specification`` 方法中。
特性描述使用字符串替代方法名称。

一个特性方法包含多个 ``block`` ，上面例子中我们使用 ``setup, when, then`` 。这里的 ``setup`` block 比较特殊，它是可选的，其允许配置本地的变量在特性方法中可见。 ``when`` block 定义执行内容，``then`` 描述执行返回结果。

注意，``StackSpec`` 中 ``setup`` 方法里有一个描述字符串。描述字符串为可选，可以下在 ``block`` 标签后添加(例如：setup ，when， then)。

More Spock
----------

``Spock`` 提供了很多特性，像数据表，高级的 ``mocking`` 功能。如果想了解更多可查看 `Spock GitHub page <https://github.com/spockframework/spock>`_

Functional Tests with Geb
-------------------------

``Geb`` 是 web 功能测试库，集成于 Junit 和 Spock。 其基于 ``Selenium`` web 驱动，和 spock 一样也提供 DSL 编写功能测试。

``Geb`` 的强大特性非常适合功能测试：

- DOM access via a JQuery-like $ function

- implements the page pattern

- support for modularization of certain web components (e.g. menu-bars, etc.) with modules

- integration with JavaScript via the JS variable

这一章节不会详细介绍如何使用 ``Geb``，但会让大家了解它，知道如果进行功能测试。

下面章节将通过例子介绍通过 ``Geb`` 为一个简单的 web page 写功能测试。

A Geb Script
^^^^^^^^^^^^

尽管 ``Geb`` 可以单独在 Groovy 脚本中使用， 但是在大多数场景下都会与其他测试框架组合使用。
``Geb`` 中各种基类可以在 Junit3,Junit4,TestNG 或 Spock  中使用。
这些基类是 Geb module 的一部分，需要添加依赖。

For example, the following @Grab dependencies have to be used to run Geb with the Selenium Firefox driver in JUnit4 tests. The module that is needed for JUnit 3/4 support is geb-junit:

.. code-block:: groovy

	@Grapes([
	    @Grab("org.gebish:geb-core:0.9.2"),
	    @Grab("org.gebish:geb-junit:0.9.2"),
	    @Grab("org.seleniumhq.selenium:selenium-firefox-driver:2.26.0"),
	    @Grab("org.seleniumhq.selenium:selenium-support:2.26.0")
	])
	
The central class in Geb is the geb.Browser class. As its name implies it is used to browse pages and access DOM elements:

def browser = new Browser(driver: new FirefoxDriver(), baseUrl: 'http://myhost:8080/myapp')  
browser.drive {
    go "/login"                        

    $("#username").text = 'John'       
    $("#password").text = 'Doe'

    $("#loginButton").click()

    assert title == "My Application - Dashboard"
}
A new Browser instance is created. In this case it uses the Selenium FirefoxDriver and sets the baseUrl.
go is used to navigate to an URL or relative URI
$ together with CSS selectors is used to access the username and password DOM fields.
The Browser class comes with a drive method that delegates all method/property calls to the current browser instance. The Browser configuration must not be done inline, it can also be externalized in a GebConfig.groovy configuration file for example. In practice, the usage of the Browser class is mostly hidden by Geb test base classes. They delegate all missing properties and method calls to the current browser instance that exists in the background:

class SearchTests extends geb.junit4.GebTest {

    @Test
    void executeSeach() {
        go 'http://somehost/mayapp/search'              
        $('#searchField').text = 'John Doe'             
        $('#searchButton').click()                      

        assert $('.searchResult a').first().text() == 'Mr. John Doe' 
    }
}
Browser#go takes a relative or absolute link and calls the page.
Browser#$ is used to access DOM content. Any CSS selectors supported by the underlying Selenium drivers are allowed
click is used to click a button.
$ is used to get the first link out of the searchResult block
The example above shows a simple Geb web test with the JUnit 4 base class geb.junit4.GebTest. Note that in this case the Browser configuration is externalized. GebTest delegates methods like go and $ to the underlying browser instance.

5.2. More Geb

In the previous section we only scratched the surface of the available Geb features. More information on Geb can be found at the project homepage.