领域专属语言 DSL
================


Command chains
------------------


``Groovy`` 允许在方法调用的参数周围省略括号。
在 ``command chain`` 中同样也可以如此省略括号，以及方法链之间的点号。
总体的思路是在调用 ``a b c d`` 与 ``a(b).c(d)`` 的含义是一致。
这同样适用于多参数，闭包参数和命名参数。
这样的方法链也可以用于赋值使用。
下面的一些例子中，可以看到这种语法的使用：

.. code-block:: groovy

	// equivalent to: turn(left).then(right)
	turn left then right

	// equivalent to: take(2.pills).of(chloroquinine).after(6.hours)
	take 2.pills of chloroquinine after 6.hours

	// equivalent to: paint(wall).with(red, green).and(yellow)
	paint wall with red, green and yellow

	// with named parameters too
	// equivalent to: check(that: margarita).tastes(good)
	check that: margarita tastes good

	// with closures as parameters
	// equivalent to: given({}).when({}).then({})
	given { } when { } then { }

在方法链中如果使用不带参数的方法，是需要添加括号的：

.. code-block:: groovy

	// equivalent to: select(all).unique().from(names)
	select all unique() from names

如果 ``command chain`` 由奇数个元素（方法、参数），	其结尾元素为属性：

.. code-block:: groovy

	// equivalent to: take(3).cookies
	// and also this: take(3).getCookies()

	take 3 cookies

``command chain`` 使得使用 ``Groovy`` 来编写更广泛的 ``DSL`` 成为可能。

上面例子，我们看到了基于 ``DSL`` 的 ``command chain`` 的使用。
下面例子中我们将介绍，第一个例子，使用 ``maps`` 和 ``closures`` 来创建一个 ``DSL``：
 
.. code-block::  groovy
 
	show = { println it }
	square_root = { Math.sqrt(it) }

	def please(action) {
	  [the: { what ->
	    [of: { n -> action(what(n)) }]
	  }]
	}

	// equivalent to: please(show).the(square_root).of(100)
	please show the square_root of 100
	// ==> 10.0

第二例子中将考虑如何通过 ``DSL`` 来简化现存的 API。
你可能需要将你的代码给客户，业务分析师，测试人员来阅读，但是他们都不会是 Java 开发人员。
这里我们使用 `Google Guava 库 <https://github.com/google/guava>`_ 中的 Splitter，尽管它已经有了非仓不错的 ``Fluent`` API，但我们还是对他做一些包装。

.. code-block::  groovy

	@Grab('com.google.guava:guava:r09')
	import com.google.common.base.*
	def result = Splitter.on(',').trimResults(CharMatcher.is('_' as char)).split("_a ,_b_ ,c__").iterator().toList()

这段代码对于 Java 开发人员非常好理解，但是对于你的目标客户就不一定了，并且也需要写很多的代码。
在这里我们还是通过使用 maps 和 closures 来让它变得更加的简单：

.. code-block:: groovy

	@Grab('com.google.guava:guava:r09')
	import com.google.common.base.*
	def split(string) {
	  [on: { sep ->
	    [trimming: { trimChar ->
	      Splitter.on(sep).trimResults(CharMatcher.is(trimChar as char)).split(string).iterator().toList()
	    }]
	  }]
	}

我们可以这样写：

.. code-block:: groovy


	def result = split "_a ,_b_ ,c__" on ',' trimming '_'


操作符重载 (Operator overloading)
-----------------------------------

``Groovy`` 中的各种操作符都对应相关的方法调用。
这样可以在 ``Groovy`` 或 ``Java`` 对象上利用操作符重载。
下表中描述 ``Groovy`` 中操作符与方法的对应关系：

+--------------------------+---------------------+
| Operator                 | Method              |
+==========================+=====================+
| a + b                    | a.plus(b)           |
+--------------------------+---------------------+
| a - b                    | a.minus(b)          |
+--------------------------+---------------------+
| a * b                    | a.multiply(b)       |
+--------------------------+---------------------+
| a ** b                   |    a.power(b)       |
+--------------------------+---------------------+
| a / b                    |    a.div(b)         |
+--------------------------+---------------------+
| a % b                    |    a.mod(b)         |
+--------------------------+---------------------+
| a | b                    |    a.or(b)          |
+--------------------------+---------------------+
| a ^ b                    |   a.xor(b)          | 
+--------------------------+---------------------+
| a++ or ++a               |    a.next()         |
+--------------------------+---------------------+
| a-- or --a               |    a.previous()     |
+--------------------------+---------------------+
|a[b]                      |    a.getAt(b)       |
+--------------------------+---------------------+
| a[b] = c                 |  a.putAt(b, c)      |
+--------------------------+---------------------+
| a << b                   |  a.leftShift(b)     |
+--------------------------+---------------------+
| a >> b                   |  a.rightShift(b)    |
+--------------------------+---------------------+
| switch(a) { case(b) : }  |  b.isCase(a)        |
+--------------------------+---------------------+
| if(a)                    |    a.asBoolean()    |
+--------------------------+---------------------+
| ~a                       |  a.bitwiseNegate()  |
+--------------------------+---------------------+
| -a                       |    a.negative()     |
+--------------------------+---------------------+
| +a                       |    a.positive()     |
+--------------------------+---------------------+
| a as b                   |    a.asType(b)      |
+--------------------------+---------------------+
| a == b                   |    a.equals(b)      |
+--------------------------+---------------------+
| a != b                   | !a.equals(b)        |
+--------------------------+---------------------+
| a <=> b                  |    a.compareTo(b)   |
+--------------------------+---------------------+
| a > b                    | a.compareTo(b) > 0  |
+--------------------------+---------------------+
| a >= b                   |a.compareTo(b) >= 0  |  
+--------------------------+---------------------+
| a < b                    |a.compareTo(b) < 0   |
+--------------------------+---------------------+
| a <= b                   |a.compareTo(b) <= 0  |
+--------------------------+---------------------+


Script base classes
------------------------

The Script Class
^^^^^^^^^^^^^^^^

``Groovy`` scripts 都会被编译为 ``classes`` 。例如:

.. code-block:: groovy

	println 'Hello from Groovy'

被编译为继承与抽象类 ``groovy.lang.Script`` 的类。这个类只有一个抽象方法 ``run`` 。当 ``script`` 完成变异，其代码体就成为 run 方法，脚本中其他方法将在实现类中。
``Script`` 类通过  ``Binding`` 对象，提供了与应用集成的基础条件，例如：

.. code-block:: groovy

	def binding = new Binding()              // <1>        
	def shell = new GroovyShell(binding)     // <2>
	binding.setVariable('x',1)               // <3>
	binding.setVariable('y',3)
	shell.evaluate 'z=2*x+y'                 // <4>
	assert binding.getVariable('z') == 5     // <5>

<1> a binding is used to share data between the script and the calling class

<2> a GroovyShell can be used with this binding

<3> input variables are set from the calling class inside the binding

<4> then the script is evaluated

<5> and the z variable has been "exported" into the binding

这是一种非常实用的方式在调用方与脚本中共享数据，然而在一些情况下其也存在一些不足的地方。
基于此，``Groovy`` 中可以自定义 ``Script`` ，通过定义抽象类继承 ``groovy.lang.Script``:

.. code-block:: groovy

	abstract class MyBaseClass extends Script {
	    String name
	    public void greet() { println "Hello, $name!" }
	}

让后将自定义的 ``	script`` 类在编译器中声明，如：

.. code-block:: groovy

	def config = new CompilerConfiguration()     		         // <1>                           
	config.scriptBaseClass = 'MyBaseClass'                       // <2>           
	def shell = new GroovyShell(this.class.classLoader, config)  // <3>           
	shell.evaluate """
	    setName 'Judith'                                         // <4>           
	    greet()
	"""

<1> create a custom compiler configuration

<2> set the base script class to our custom base script class

<3> then create a GroovyShell using that configuration

<4> the script will then extend the base script class, giving direct access to the name property and greet method


The @BaseScript annotation
^^^^^^^^^^^^^^^^^^^^^^^^^^

这里还可以使用 ``@BaseScript`` 注释：

.. code-block:: groovy

	import groovy.transform.BaseScript

	@BaseScript MyBaseClass baseScript
	setName 'Judith'
	greet()

``@BaseScript`` 需要在注释在自定义的脚本类型变量上，或者将自定义脚本类型作为 ``BaseScript`` 的成员，在执行脚本上注释：

	@BaseScript(MyBaseClass)
	import groovy.transform.BaseScript

	setName 'Judith'
	greet()


Alternate abstract method
^^^^^^^^^^^^^^^^^^^^^^^^^
我们知道了在 ``script`` 类中只有一个需要实现的抽象方法 ``run`` 。 ``run`` 方法在脚本引擎中自动执行。在有些情况下，基类中
实现了 run 方法，并提供了一个替代方法供脚本使用。下面例子中，run 方法用于初始化：

.. code-block:: groovy

	abstract class MyBaseClass extends Script {
	    int count
	    abstract void scriptBody()                    // <1>                      
	    def run() { 
	        count++                                   // <2>          
	        scriptBody()                              // <3>         
	        count                                     // <4>         
	    }
	}

<1> the base script class should define one (and only one) abstract method

<2> the run method can be overriden and perform a task before executing the script body

<3> run calls the abstract scriptBody method which will delegate to the user script

<4> then it can return something else than the value from the script

If you execute this code:

.. code-block:: groovy

	def result = shell.evaluate """
	    println 'Ok'
	"""
	assert result == 1

你会看到脚本执行后，run 方法将返回 ``1``。
使用 ``parse`` 替代 ``evaluate`` 执行，将会更加清晰，这样可以在同一脚本上执行多次 run 方法：

.. code-block:: groovy

	def script = shell.parse("println 'Ok'")
	assert script.run() == 1
	assert script.run() == 2


Adding properties to numbers
----------------------------	

In Groovy number types are considered equal to any other types.

这样可以通过添加属性或方法来增强 ``numbers``。这样在处理数量计算上会非常便利。详细的增强方式可以查看章节 `extension modules <http://docs.groovy-lang.org/latest/html/documentation/core-metaprogramming.html#_extension_modules>`_ 和 `categories <http://docs.groovy-lang.org/latest/html/documentation/core-metaprogramming.html#categories>`_

这里我们使用 ``TimeCategory``:
 
.. code-block:: groovy

	use(TimeCategory)  {
	    println 1.minute.from.now       		// <1>
	    println 10.hours.ago

	    def someDate = new Date()               // <2> 
	    println someDate - 3.months
	}

<1> using the TimeCategory, a property minute is added to the Integer class

<2> similarily, the months method returns a groovy.time.DatumDependentDuration which can be used in calculus

``Categories`` 是词汇绑定，使得其非常适合定义内部 DSL.


@DelegatesTo
------------

编译期代理策略
^^^^^^^^^^^^^^^^^^

@groovy.lang.DelegatesTo is a documentation and compile-time annotation aimed at:

documenting APIs that use closures as arguments

providing type information for the static type checker and compiler

The Groovy language is a platform of choice for building DSLs. Using closures, it’s quite easy to create custom control structures, as well as it is simple to create builders. Imagine that you have the following code:

``Groovy`` 语言是平台选择来构建 ``DSLs``。使用闭包可以很容易的创建自定义的结构。 想象下面的代码：

.. code-block:: groovy

	email {
	    from 'dsl-guru@mycompany.com'
	    to 'john.doe@waitaminute.com'
	    subject 'The pope has resigned!'
	    body {
	        p 'Really, the pope has resigned!'
	    }
	}

使用构建者策略来实现，使用命名为 ``email`` 方法来接收	一个闭包参数。该方法将代理后续方法（from , to, subject, body）的调用。
body 作为一个方法接收闭包参数，同样也使用构建者策略。

实现构建者通常可以使用下面方式：

.. code-block:: groovy

	def email(Closure cl) {
	    def email = new EmailSpec()
	    def code = cl.rehydrate(email, this, this)
	    code.resolveStrategy = Closure.DELEGATE_ONLY
	    code()
	}

``EmailSpec`` 实现了 ``from, to, ...`` 一系列方法。调用 ``rehydrate`` 拷贝一份闭包用于设置 ``delegate`` , ``owner`` 以及 ``thisObject`` 。
当设置 ``DELEGATE_ONLY`` 时，设置 owner 与 this 就不是很重要，这种情况下只处理闭包的代理。

.. code-block:: groovy

	class EmailSpec {
	    void from(String from) { println "From: $from"}
	    void to(String... to) { println "To: $to"}
	    void subject(String subject) { println "Subject: $subject"}
	    void body(Closure body) {
	        def bodySpec = new BodySpec()
	        def code = body.rehydrate(bodySpec, this, this)
	        code.resolveStrategy = Closure.DELEGATE_ONLY
	        code()
	    }
	}

``EmailSpec`` 中 body 方法接收一个闭包，先拷贝，再执行。
这种方式在 ``Groovy`` 中被称为构建者模式。

One of the problems with the code that we’ve shown is that the user of the email method doesn’t have any information about the methods that he’s allowed to call inside the closure. The only possible information is from the method documentation. There are two issues with this: first of all, documentation is not always written, and if it is, it’s not always available (javadoc not downloaded, for example). Second, it doesn’t help IDEs. What would be really interesting, here, is for IDEs to help the developer by suggesting, once they are in the closure body, methods that exist on the email class.

Moreover, if the user calls a method in the closure which is not defined by the EmailSpec class, the IDE should at least issue a warning (because it’s very likely that it will break at runtime).

One more problem with the code above is that it is not compatible with static type checking. Type checking would let the user know if a method call is authorized at compile time instead of runtime, but if you try to perform type checking on this code:

.. code-block:: groovy

	email {
	    from 'dsl-guru@mycompany.com'
	    to 'john.doe@waitaminute.com'
	    subject 'The pope has resigned!'
	    body {
	        p 'Really, the pope has resigned!'
	    }
	}

Then the type checker will know that there’s an email method accepting a Closure, but it will complain for every method call inside the closure, because from, for example, is not a method which is defined in the class. Indeed, it’s defined in the EmailSpec class and it has absolutely no hint to help it knowing that the closure delegate will, at runtime, be of type EmailSpec:

.. code-block:: groovy

	@groovy.transform.TypeChecked
	void sendEmail() {
	    email {
	        from 'dsl-guru@mycompany.com'
	        to 'john.doe@waitaminute.com'
	        subject 'The pope has resigned!'
	        body {
	            p 'Really, the pope has resigned!'
	        }
	    }
	}

will fail compilation with errors like this one:

.. note:: 

	[Static type checking] - Cannot find matching method MyScript#from(java.lang.String). Please check if the declared type is right and if the method exists.
	 @ line 31, column 21.
	                       from 'dsl-guru@mycompany.com'


@DelegatesTo
^^^^^^^^^^^^^^^

For those reasons, Groovy 2.1 introduced a new annotation named @DelegatesTo. The goal of this annotation is to solve both the documentation issue, that will let your IDE know about the expected methods in the closure body, and it will also solve the type checking issue, by giving hints to the compiler about what are the potential receivers of method calls in the closure body.

The idea is to annotate the Closure parameter of the email method:

.. code-block:: groovy

	def email(@DelegatesTo(EmailSpec) Closure cl) {
	    def email = new EmailSpec()
	    def code = cl.rehydrate(email, this, this)
	    code.resolveStrategy = Closure.DELEGATE_ONLY
	    code()
	}

What we’ve done here is telling the compiler (or the IDE) that when the method will be called with a closure, the delegate of this closure will be set to an object of type email. But there is still a problem: the default delegation strategy is not the one which is used in our method. So we will give more information and tell the compiler (or the IDE) that the delegation strategy is also changed:

.. code-block:: groovy

	def email(@DelegatesTo(strategy=Closure.DELEGATE_ONLY, value=EmailSpec) Closure cl) {
	    def email = new EmailSpec()
	    def code = cl.rehydrate(email, this, this)
	    code.resolveStrategy = Closure.DELEGATE_ONLY
	    code()
	}

Now, both the IDE and the type checker (if you are using @TypeChecked) will be aware of the delegate and the delegation strategy. This is very nice because it will both allow the IDE to provide smart completion, but it will also remove errors at compile time that exist only because the behaviour of the program is normally only known at runtime!

The following code will now pass compilation:

.. code-block:: groovy

	@TypeChecked
	void doEmail() {
	    email {
	        from 'dsl-guru@mycompany.com'
	        to 'john.doe@waitaminute.com'
	        subject 'The pope has resigned!'
	        body {
	            p 'Really, the pope has resigned!'
	        }
	    }
	}


DelegatesTo modes
^^^^^^^^^^^^^^^^^


@DelegatesTo supports multiple modes that we will describe with examples in this section.


Simple delegation
""""""""""""""""""""""


In this mode, the only mandatory parameter is the value which says to which class we delegate calls. Nothing more. We’re telling the compiler that the type of the delegate will always be of the type documented by @DelegatesTo (note that it can be a subclass, but if it is, the methods defined by the subclass will not be visible to the type checker).

.. code-block:: groovy

	void body(@DelegatesTo(BodySpec) Closure cl) {
	    // ...
	}


Delegation strategy
"""""""""""""""""""""

In this mode, you must specify both the delegate class and a delegation strategy. This must be used if the closure will not be called with the default delegation strategy, which is Closure.OWNER_FIRST.

.. code-block:: groovy

	void body(@DelegatesTo(strategy=Closure.DELEGATE_ONLY, value=BodySpec) Closure cl) {
	    // ...
	}


Delegate to parameter
""""""""""""""""""""""


In this variant, we will tell the compiler that we are delegating to another parameter of the method. Take the following code:

.. code-block:: groovy

	def exec(Object target, Closure code) {
	   def clone = code.rehydrate(target, this, this)
	   clone()
	}

Here, the delegate which will be used is not created inside the exec method. In fact, we take an argument of the method and delegate to it. Usage may look like this:

.. code-block:: groovy

	def email = new Email()
	exec(email) {
	   from '...'
	   to '...'
	   send()
	}

Each of the method calls are delegated to the email parameter. This is a widely used pattern which is also supported by @DelegatesTo using a companion annotation:

.. code-block:: groovy

	def exec(@DelegatesTo.Target Object target, @DelegatesTo Closure code) {
	   def clone = code.rehydrate(target, this, this)
	   clone()
	}

A closure is annotated with @DelegatesTo, but this time, without specifying any class. Instead, we’re annotating another parameter with @DelegatesTo.Target. The type of the delegate is then determined at compile time. One could think that we are using the parameter type, which in this case is Object but this is not true. Take this code:

.. code-block:: groovy

	class Greeter {
	   void sayHello() { println 'Hello' }
	}
	def greeter = new Greeter()
	exec(greeter) {
	   sayHello()
	}

Remember that this works out of the box without having to annotate with @DelegatesTo. However, to make the IDE aware of the delegate type, or the type checker aware of it, we need to add @DelegatesTo. And in this case, it will now that the Greeter variable is of type Greeter, so it will not report errors on the sayHello method even if the exec method doesn’t explicitly define the target as of type Greeter. This is a very powerful feature, because it prevents you from writing multiple versions of the same exec method for different receiver types!

In this mode, the @DelegatesTo annotation also supports the strategy parameter that we’ve described upper.

5.3.4. Multiple closures

In the previous example, the exec method accepted only one closure, but you may have methods that take multiple closures:

void fooBarBaz(Closure foo, Closure bar, Closure baz) {
    ...
}
Then nothing prevents you from annotating each closure with @DelegatesTo:

class Foo { void foo(String msg) { println "Foo ${msg}!" } }
class Bar { void bar(int x) { println "Bar ${x}!" } }
class Baz { void baz(Date d) { println "Baz ${d}!" } }

void fooBarBaz(@DelegatesTo(Foo) Closure foo, @DelegatesTo(Bar) Closure bar, @DelegatesTo(Baz) Closure baz) {
   ...
}
But more importantly, if you have multiple closures and multiple arguments, you can use several targets:

void fooBarBaz(
    @DelegatesTo.Target('foo') foo,
    @DelegatesTo.Target('bar') bar,
    @DelegatesTo.Target('baz') baz,

    @DelegatesTo(target='foo') Closure cl1,
    @DelegatesTo(target='bar') Closure cl2,
    @DelegatesTo(target='baz') Closure cl3) {
    cl1.rehydrate(foo, this, this).call()
    cl2.rehydrate(bar, this, this).call()
    cl3.rehydrate(baz, this, this).call()
}

def a = new Foo()
def b = new Bar()
def c = new Baz()
fooBarBaz(
    a, b, c,
    { foo('Hello') },
    { bar(123) },
    { baz(new Date()) }
)
At this point, you may wonder why we don’t use the parameter names as references. The reason is that the information (the parameter name) is not always available (it’s a debug-only information), so it’s a limitation of the JVM.
5.3.5. Delegating to a generic type

In some situations, it is interesting to instruct the IDE or the compiler that the delegate type will not be a parameter but a generic type. Imagine a configurator that runs on a list of elements:

public <T> void configure(List<T> elements, Closure configuration) {
   elements.each { e->
      def clone = configuration.rehydrate(e, this, this)
      clone.resolveStrategy = Closure.DELEGATE_FIRST
      clone.call()
   }
}
Then this method can be called with any list like this:

@groovy.transform.ToString
class Realm {
   String name
}
List<Realm> list = []
3.times { list << new Realm() }
configure(list) {
   name = 'My Realm'
}
assert list.every { it.name == 'My Realm' }
To let the type checker and the IDE know that the configure method calls the closure on each element of the list, you need to use @DelegatesTo differently:

public <T> void configure(
    @DelegatesTo.Target List<T> elements,
    @DelegatesTo(strategy=Closure.DELEGATE_FIRST, genericTypeIndex=0) Closure configuration) {
   def clone = configuration.rehydrate(e, this, this)
   clone.resolveStrategy = Closure.DELEGATE_FIRST
   clone.call()
}
@DelegatesTo takes an optional genericTypeIndex argument that tells what is the index of the generic type that will be used as the delegate type. This must be used in conjunction with @DelegatesTo.Target and the index starts at 0. In the example above, that means that the delegate type is resolved against List<T>, and since the generic type at index 0 is T and inferred as a Realm, the type checker infers that the delegate type will be of type Realm.

We’re using a genericTypeIndex instead of a placeholder (T) because of JVM limitations.
5.3.6. Delegating to an arbitrary type

It is possible that none of the options above can represent the type you want to delegate to. For example, let’s define a mapper class which is parametrized with an object and defines a map method which returns an object of another type:

class Mapper<T,U> {                             
    final T value                               
    Mapper(T value) { this.value = value }
    U map(Closure<U> producer) {                
        producer.delegate = value
        producer()
    }
}
The mapper class takes two generic type arguments: the source type and the target type
The source object is stored in a final field
The map method asks to convert the source object to a target object
As you can see, the method signature from map does not give any information about what object will be manipulated by the closure. Reading the method body, we know that it will be the value which is of type T, but T is not found in the method signature, so we are facing a case where none of the available options for @DelegatesTo is suitable. For example, if we try to statically compile this code:

def mapper = new Mapper<String,Integer>('Hello')
assert mapper.map { length() } == 5
Then the compiler will fail with:

Static type checking] - Cannot find matching method TestScript0#length()
In that case, you can use the type member of the @DelegatesTo annotation to reference T as a type token:

class Mapper<T,U> {
    final T value
    Mapper(T value) { this.value = value }
    U map(@DelegatesTo(type="T") Closure<U> producer) {  
        producer.delegate = value
        producer()
    }
}
The @DelegatesTo annotation references a generic type which is not found in the method signature
Note that you are not limited to generic type tokens. The type member can be used to represent complex types, such as List<T> or Map<T,List<U>>. The reason why you should use that in last resort is that the type is only checked when the type checker finds usage of @DelegatesTo, not when the annotated method itself is compiled. This means that type safety is only ensured at the call site. Additionally, compilation will be slower (though probably unnoticeable for most cases).

6. Compilation customizers

6.1. Introduction

Whether you are using groovyc to compile classes or a GroovyShell, for example, to execute scripts, under the hood, a compiler configuration is used. This configuration holds information like the source encoding or the classpath but it can also be used to perform more operations like adding imports by default, applying AST transformations transparently or disabling global AST transformations.

The goal of compilation customizers is to make those common tasks easy to implement. For that, the CompilerConfiguration class is the entry point. The general schema will always be based on the following code:

import org.codehaus.groovy.control.CompilerConfiguration
// create a configuration
def config = new CompilerConfiguration()
// tweak the configuration
config.addCompilationCustomizers(...)
// run your script
def shell = new GroovyShell(config)
shell.evaluate(script)
Compilation customizers must extend the org.codehaus.groovy.control.customizers.CompilationCustomizer class. A customizer works:

on a specific compilation phase

on every class node being compiled

You can implement your own compilation customizer but Groovy includes some of the most common operations.

6.2. Import customizer

Using this compilation customizer, your code will have imports added transparently. This is in particular useful for scripts implementing a DSL where you want to avoid users from having to write imports. The import customizer will let you add all the variants of imports the Groovy language allows, that is:

class imports, optionally aliased

star imports

static imports, optionally aliased

static star imports

import org.codehaus.groovy.control.customizers.ImportCustomizer

def icz = new ImportCustomizer()
// "normal" import
icz.addImports('java.util.concurrent.atomic.AtomicInteger', 'java.util.concurrent.ConcurrentHashMap')
// "aliases" import
icz.addImport('CHM', 'java.util.concurrent.ConcurrentHashMap')
// "static" import
icz.addStaticImport('java.lang.Math', 'PI') // import static java.lang.Math.PI
// "aliased static" import
icz.addStaticImport('pi', 'java.lang.Math', 'PI') // import static java.lang.Math.PI as pi
// "star" import
icz.addStarImports 'java.util.concurrent' // import java.util.concurrent.*
// "static star" import
icz.addStaticStars 'java.lang.Math' // import static java.lang.Math.*
A detailed description of all shortcuts can be found in org.codehaus.groovy.control.customizers.ImportCustomizer

6.3. AST transformation customizer

The AST transformation customizer is meant to apply AST transformations transparently. Unlike global AST transformations that apply on every class beeing compiled as long as the transform is found on classpath (which has drawbacks like increasing the compilation time or side effects due to transformations applied where they should not), the customizer will allow you to selectively apply a transform only for specific scripts or classes.

As an example, let’s say you want to be able to use @Log in a script. The problem is that @Log is normally applied on a class node and a script, by definition, doesn’t require one. But implementation wise, scripts are classes, it’s just that you cannot annotate this implicit class node with @Log. Using the AST customizer, you have a workaround to do it:

import org.codehaus.groovy.control.customizers.ASTTransformationCustomizer
import groovy.util.logging.Log

def acz = new ASTTransformationCustomizer(Log)
config.addCompilationCustomizers(acz)
That’s all! Internally, the @Log AST transformation is applied to every class node in the compilation unit. This means that it will be applied to the script, but also to classes defined within the script.

If the AST transformation that you are using accepts parameters, you can use parameters in the constructor too:

def acz = new ASTTransformationCustomizer(Log, value: 'LOGGER')
// use name 'LOGGER' instead of the default 'log'
config.addCompilationCustomizers(acz)
As the AST transformation customizers works with objects instead of AST nodes, not all values can be converted to AST transformation parameters. For example, primitive types are converted to ConstantExpression (that is LOGGER is converted to new ConstantExpression('LOGGER'), but if your AST transformation takes a closure as an argument, then you have to give it a ClosureExpression, like in the following example:

def configuration = new CompilerConfiguration()
def expression = new AstBuilder().buildFromCode(CompilePhase.CONVERSION) { -> true }.expression[0]
def customizer = new ASTTransformationCustomizer(ConditionalInterrupt, value: expression, thrown: SecurityException)
configuration.addCompilationCustomizers(customizer)
def shell = new GroovyShell(configuration)
shouldFail(SecurityException) {
    shell.evaluate("""
        // equivalent to adding @ConditionalInterrupt(value={true}, thrown: SecurityException)
        class MyClass {
            void doIt() { }
        }
        new MyClass().doIt()
    """)
}
For a complete list of options, please refer to org.codehaus.groovy.control.customizers.ASTTransformationCustomizer

6.4. Secure AST customizer

This customizer will allow the developer of a DSL to restrict the grammar of the language, to prevent users from using some constructs, for example. It is only ``secure'' in that sense only and it is very important to understand that it does not replace a security manager. The only reason for it to exist is to limit the expressiveness of the language. This customizer only works at the AST (abstract syntax tree) level, not at runtime! It can be strange at first glance, but it makes much more sense if you think of Groovy as a platform to build DSLs. You may not want a user to have a complete language at hand. In the example below, we will demonstrate it using an example of language that only allows arithmetic operations, but this customizer allows you to:

allow/disallow creation of closures

allow/disallow imports

allow/disallow package definition

allow/disallow definition of methods

restrict the receivers of method calls

restrict the kind of AST expressions a user can use

restrict the tokens (grammar-wise) a user can use

restrict the types of the constants that can be used in code

For all those features, the secure AST customizer works using either a whitelist (list of elements that are allowed) or a blacklist (list of elements that are disallowed). For each type of feature (imports, tokens, …) you have the choice to use either a whitelist or a blacklist, but you can mix whitelists and blacklists for distinct features. In general, you will choose whitelists (disallow all, allow selected).

import org.codehaus.groovy.control.customizers.SecureASTCustomizer
import static org.codehaus.groovy.syntax.Types.* 

def scz = new SecureASTCustomizer()
scz.with {
    closuresAllowed = false // user will not be able to write closures
    methodDefinitionAllowed = false // user will not be able to define methods
    importsWhitelist = [] // empty whitelist means imports are disallowed
    staticImportsWhitelist = [] // same for static imports
    staticStarImportsWhitelist = ['java.lang.Math'] // only java.lang.Math is allowed
    // the list of tokens the user can find
    // constants are defined in org.codehaus.groovy.syntax.Types
    tokensWhitelist = [ 
            PLUS,
            MINUS,
            MULTIPLY,
            DIVIDE,
            MOD,
            POWER,
            PLUS_PLUS,
            MINUS_MINUS,
            COMPARE_EQUAL,
            COMPARE_NOT_EQUAL,
            COMPARE_LESS_THAN,
            COMPARE_LESS_THAN_EQUAL,
            COMPARE_GREATER_THAN,
            COMPARE_GREATER_THAN_EQUAL,
    ].asImmutable()
    // limit the types of constants that a user can define to number types only
    constantTypesClassesWhiteList = [ 
            Integer,
            Float,
            Long,
            Double,
            BigDecimal,
            Integer.TYPE,
            Long.TYPE,
            Float.TYPE,
            Double.TYPE
    ].asImmutable()
    // method calls are only allowed if the receiver is of one of those types
    // be careful, it's not a runtime type!
    receiversClassesWhiteList = [ 
            Math,
            Integer,
            Float,
            Double,
            Long,
            BigDecimal
    ].asImmutable()
}
use for token types from org.codehaus.groovy.syntax.Types
you can use class literals here
If what the secure AST customizer provides out of the box isn’t enough for your needs, before creating your own compilation customizer, you might be interested in the expression and statement checkers that the AST customizer supports. Basically, it allows you to add custom checks on the AST tree, on expressions (expression checkers) or statements (statement checkers). For this, you must implement org.codehaus.groovy.control.customizers.SecureASTCustomizer.StatementChecker or org.codehaus.groovy.control.customizers.SecureASTCustomizer.ExpressionChecker.

Those interfaces define a single method called isAuthorized, returning a boolean, and taking a Statement (or Expression) as a parameter. It allows you to perform complex logic over expressions or statements to tell if a user is allowed to do it or not.

For example, there’s no predefined configuration flag in the customizer which will let you prevent people from using an attribute expression. Using a custom checker, it is trivial:

def scz = new SecureASTCustomizer()
def checker = { expr ->
    !(expr instanceof AttributeExpression)
} as SecureASTCustomizer.ExpressionChecker
scz.addExpressionCheckers(checker)
Then we can make sure that this works by evaluating a simple script:

new GroovyShell(config).evaluate '''
    class A {
        int val
    }
    
    def a = new A(val: 123)
    a.@val 
'''
will fail compilation
Statements can be checked using org.codehaus.groovy.control.customizers.SecureASTCustomizer.StatementChecker Expressions can be checked using org.codehaus.groovy.control.customizers.SecureASTCustomizer.ExpressionChecker

6.5. Source aware customizer

This customizer may be used as a filter on other customizers. The filter, in that case, is the org.codehaus.groovy.control.SourceUnit. For this, the source aware customizer takes another customizer as a delegate, and it will apply customization of that delegate only and only if predicates on the source unit match.

SourceUnit gives you access to multiple things but in particular the file being compiled (if compiling from a file, of course). It gives you the potential to perform operation based on the file name, for example. Here is how you would create a source aware customizer:

import org.codehaus.groovy.control.customizers.SourceAwareCustomizer
import org.codehaus.groovy.control.customizers.ImportCustomizer

def delegate = new ImportCustomizer()
def sac = new SourceAwareCustomizer(delegate)
Then you can use predicates on the source aware customizer:

// the customizer will only be applied to classes contained in a file name ending with 'Bean'
sac.baseNameValidator = { baseName ->
    baseName.endsWith 'Bean'
}

// the customizer will only be applied to files which extension is '.spec'
sac.extensionValidator = { ext -> ext == 'spec' }

// source unit validation
// allow compilation only if the file contains at most 1 class
sac.sourceUnitValidator = { SourceUnit sourceUnit -> sourceUnit.AST.classes.size() == 1 }

// class validation
// the customizer will only be applied to classes ending with 'Bean'
sac.classValidator = { ClassNode cn -> cn.endsWith('Bean') }
6.6. Customizer builder

If you are using compilation customizers in Groovy code (like the examples above) then you can use an alternative syntax to customize compilation. A builder (org.codehaus.groovy.control.customizers.builder.CompilerCustomizationBuilder) simplifies the creation of customizers using a hierarchical DSL.

import org.codehaus.groovy.control.CompilerConfiguration
import static org.codehaus.groovy.control.customizers.builder.CompilerCustomizationBuilder.withConfig 

def conf = new CompilerConfiguration()
withConfig(conf) {
    // ... 
}
static import of the builder method
configuration goes here
The code sample above shows how to use the builder. A static method, withConfig, takes a closure corresponding to the builder code, and automatically registers compilation customizers to the configuration. Every compilation customizer available in the distribution can be configured this way:

6.6.1. Import customizer

withConfig(configuration) {
   imports { // imports customizer
      normal 'my.package.MyClass' // a normal import
      alias 'AI', 'java.util.concurrent.atomic.AtomicInteger' // an aliased import
      star 'java.util.concurrent' // star imports
      staticMember 'java.lang.Math', 'PI' // static import
      staticMember 'pi', 'java.lang.Math', 'PI' // aliased static import
   }
}
6.6.2. AST transformation customizer

withConfig(conf) {
   ast(Log) 
}

withConfig(conf) {
   ast(Log, value: 'LOGGER') 
}
apply @Log transparently
apply @Log with a different name for the logger
6.6.3. Secure AST customizer

withConfig(conf) {
   secureAst {
       closuresAllowed = false
       methodDefinitionAllowed = false
   }
}
6.6.4. Source aware customizer

withConfig(configuration){
    source(extension: 'sgroovy') {
        ast(CompileStatic) 
    }
}

withConfig(configuration){
    source(extensions: ['sgroovy','sg']) {
        ast(CompileStatic) 
    }
}

withConfig(configuration) {
    source(extensionValidator: { it.name in ['sgroovy','sg']}) {
        ast(CompileStatic) 
    }
}

withConfig(configuration) {
    source(basename: 'foo') {
        ast(CompileStatic) 
    }
}

withConfig(configuration) {
    source(basenames: ['foo', 'bar']) {
        ast(CompileStatic) 
    }
}

withConfig(configuration) {
    source(basenameValidator: { it in ['foo', 'bar'] }) {
        ast(CompileStatic) 
    }
}

withConfig(configuration) {
    source(unitValidator: { unit -> !unit.AST.classes.any { it.name == 'Baz' } }) {
        ast(CompileStatic) 
    }
}
apply CompileStatic AST annotation on .sgroovy files
apply CompileStatic AST annotation on .sgroovy or .sg files
apply CompileStatic AST annotation on files whose name is 'foo'
apply CompileStatic AST annotation on files whose name is 'foo' or 'bar'
apply CompileStatic AST annotation on files that do not contain a class named 'Baz'
6.6.5. Inlining a customizer

Inlined customizer allows you to write a compilation customizer directly, without having to create a class for it.

withConfig(configuration) {
    inline(phase:'CONVERSION') { source, context, classNode ->  
        println "visiting $classNode"                           
    }
}
define an inlined customizer which will execute at the CONVERSION phase
prints the name of the class node being compiled
6.6.6. Multiple customizers

Of course, the builder allows you to define multiple customizers at once:

withConfig(configuration) {
   ast(ToString)
   ast(EqualsAndHashCode)
}
6.7. Config script flag

So far, we have described how you can customize compilation using a CompilationConfiguration class, but this is only possible if you embed Groovy and that you create your own instances of CompilerConfiguration (then use it to create a GroovyShell, GroovyScriptEngine, …).

If you want it to be applied on the classes you compile with the normal Groovy compiler (that is to say with  groovyc, ant or gradle, for example), it is possible to use a compilation flag named configscript that takes a Groovy configuration script as argument.

This script gives you access to the CompilerConfiguration instance before the files are compiled (exposed into the configuration script as a variable named configuration), so that you can tweak it.

It also transparently integrates the compiler configuration builder above. As an example, let’s see how you would activate static compilation by default on all classes.

6.7.1. Static compilation by default

Normally, classes in Groovy are compiled with a dynamic runtime. You can activate static compilation by placing an annotation named @CompileStatic on any class. Some people would like to have this mode activated by default, that is to say not having to annotated classes. Using configscript, this is possible. First of all, you need to create a file named config.groovy into src/conf with the following contents:

withConfig(configuration) { 
   ast(groovy.transform.CompileStatic)
}
configuration references a CompilerConfiguration instance
That is actually all you need. You don’t have to import the builder, it’s automatically exposed in the script. Then, compile your files using the following command line:

groovyc -configscript src/conf/config.groovy src/main/groovy/MyClass.groovy
We strongly recommend you to separate configuration files from classes, hence why we suggest using the src/main and src/conf directories above.

6.8. AST transformations

If:

runtime metaprogramming doesn’t allow you do do what you want

you need to improve the performance of the execution of your DSLs

you want to leverage the same syntax as Groovy but with different semantics

you want to improve support for type checking in your DSLs

Then AST transformations are the way to go. Unlike the techniques used so far, AST transformations are meant to change or generate code before it is compiled to bytecode. AST transformations are capable of adding new methods at compile time for example, or totally changing the body of a method based on your needs. They are a very powerful tool but also come at the price of not being easy to write. For more information about AST transformations, please take a look at the compile-time metaprogramming section of this manual.

7. Custom type checking extensions

It may be interesting, in some circumstances, to provide feedback about wrong code to the user as soon as possible, that is to say when the DSL script is compiled, rather than having to wait for the execution of the script. However, this is not often possible with dynamic code. Groovy actually provides a practical answer to this known as type checking extensions.

8. Builders

(TBD)

8.1. Creating a builder

(TBD)

8.1.1. BuilderSupport

(TBD)

8.1.2. FactoryBuilderSupport

(TBD)

8.2. Existing builders

(TBD)

8.2.1. MarkupBuilder

See Creating Xml - MarkupBuilder.

8.2.2. StreamingMarkupBuilder

See Creating Xml - StreamingMarkupBuilder.

8.2.3. SaxBuilder

A builder for generating Simple API for XML (SAX) events.

If you have the following SAX handler:

class LogHandler extends org.xml.sax.helpers.DefaultHandler {

    String log = ''

    void startElement(String uri, String localName, String qName, org.xml.sax.Attributes attributes) {
        log += "Start Element: $localName, "
    }

    void endElement(String uri, String localName, String qName) {
        log += "End Element: $localName, "
    }
}
You can use SaxBuilder to generate SAX events for the handler like this:

def handler = new LogHandler()
def builder = new groovy.xml.SAXBuilder(handler)

builder.root() {
    helloWorld()
}
And then check that everything worked as expected:

assert handler.log == 'Start Element: root, Start Element: helloWorld, End Element: helloWorld, End Element: root, '
8.2.4. StaxBuilder

A Groovy builder that works with Streaming API for XML (StAX) processors.

Here is a simple example using the StAX implementation of Java to generate XML:

def factory = javax.xml.stream.XMLOutputFactory.newInstance()
def writer = new StringWriter()
def builder = new groovy.xml.StaxBuilder(factory.createXMLStreamWriter(writer))

builder.root(attribute:1) {
    elem1('hello')
    elem2('world')
}

assert writer.toString() == '<?xml version="1.0" ?><root attribute="1"><elem1>hello</elem1><elem2>world</elem2></root>'
An external library such as Jettison can be used as follows:

@Grab('org.codehaus.jettison:jettison:1.3.3')
import org.codehaus.jettison.mapped.*

def writer = new StringWriter()
def mappedWriter = new MappedXMLStreamWriter(new MappedNamespaceConvention(), writer)
def builder = new groovy.xml.StaxBuilder(mappedWriter)

builder.root(attribute:1) {
     elem1('hello')
     elem2('world')
}

assert writer.toString() == '{"root":{"@attribute":"1","elem1":"hello","elem2":"world"}}'
8.2.5. DOMBuilder

A builder for parsing HTML, XHTML and XML into a W3C DOM tree.

For example this XML String:

String recordsXML = '''
    <records>
      <car name='HSV Maloo' make='Holden' year='2006'>
        <country>Australia</country>
        <record type='speed'>Production Pickup Truck with speed of 271kph</record>
      </car>
      <car name='P50' make='Peel' year='1962'>
        <country>Isle of Man</country>
        <record type='size'>Smallest Street-Legal Car at 99cm wide and 59 kg in weight</record>
      </car>
      <car name='Royale' make='Bugatti' year='1931'>
        <country>France</country>
        <record type='price'>Most Valuable Car at $15 million</record>
      </car>
    </records>'''
Can be parsed into a DOM tree with a DOMBuilder like this:

def reader = new StringReader(recordsXML)
def doc = groovy.xml.DOMBuilder.parse(reader)
And then processed further e.g. by using DOMCategory:

def records = doc.documentElement
use(groovy.xml.dom.DOMCategory) {
    assert records.car.size() == 3
}
8.2.6. NodeBuilder

NodeBuilder is used for creating nested trees of Node objects for handling arbitrary data. To create a simple user list you use a NodeBuilder like this:

def nodeBuilder = new NodeBuilder()
def userlist = nodeBuilder.userlist {
    user(id: '1', firstname: 'John', lastname: 'Smith') {
        address(type: 'home', street: '1 Main St.', city: 'Springfield', state: 'MA', zip: '12345')
        address(type: 'work', street: '2 South St.', city: 'Boston', state: 'MA', zip: '98765')
    }
    user(id: '2', firstname: 'Alice', lastname: 'Doe')
}
Now you can process the data further, e.g. by using GPath expressions:

assert userlist.user.@firstname.join(', ') == 'John, Alice'
assert userlist.user.find { it.@lastname == 'Smith' }.address.size() == 2
8.2.7. JsonBuilder

Groovys JsonBuilder makes it easy to create Json. For example to create this Json string:

String carRecords = '''
    {
        "records": {
        "car": {
            "name": "HSV Maloo",
            "make": "Holden",
            "year": 2006,
            "country": "Australia",
            "record": {
              "type": "speed",
              "description": "production pickup truck with speed of 271kph"
            }
          }
      }
    }
'''
you can use a JsonBuilder like this:

JsonBuilder builder = new JsonBuilder()
builder.records {
  car {
        name 'HSV Maloo'
        make 'Holden'
        year 2006
        country 'Australia'
        record {
            type 'speed'
            description 'production pickup truck with speed of 271kph'
        }
  }
}
String json = JsonOutput.prettyPrint(builder.toString())
We use JsonUnit to check that the builder produced the expected result:

JsonAssert.assertJsonEquals(json, carRecords)
8.2.8. StreamingJsonBuilder

Unlike JsonBuilder which creates a data structure in memory, which is handy in those situations where you want to alter the structure programmatically before output, StreamingJsonBuilder directly streams to a writer without any intermediate memory data structure. If you do not need to modify the structure and want a more memory-efficient approach, use StreamingJsonBuilder.

The usage of StreamingJsonBuilder is similar to JsonBuilder. In order to create this Json string:

String carRecords = '''
    {
        "records": {
        "car": {
            "name": "HSV Maloo",
            "make": "Holden",
            "year": 2006,
            "country": "Australia",
            "record": {
              "type": "speed",
              "description": "production pickup truck with speed of 271kph"
            }
          }
      }
    }
'''
you use a StreamingJsonBuilder like this:

StringWriter writer = new StringWriter()
StreamingJsonBuilder builder = new StreamingJsonBuilder(writer)
builder.records {
  car {
        name 'HSV Maloo'
        make 'Holden'
        year 2006
        country 'Australia'
        record {
            type 'speed'
            description 'production pickup truck with speed of 271kph'
        }
  }
}
String json = JsonOutput.prettyPrint(writer.toString())
We use JsonUnit to check the expected result:

JsonAssert.assertJsonEquals(json, carRecords)
8.2.9. SwingBuilder

SwingBuilder allows you to create full-fledged Swing GUIs in a declarative and concise fashion. It accomplishes this by employing a common idiom in Groovy, builders. Builders handle the busywork of creating complex objects for you, such as instantiating children, calling Swing methods, and attaching these children to their parents. As a consequence, your code is much more readable and maintainable, while still allowing you access to the full range of Swing components.

Here’s a simple example of using SwingBuilder:

import groovy.swing.SwingBuilder
import java.awt.BorderLayout as BL

count = 0
new SwingBuilder().edt {
  frame(title: 'Frame', size: [300, 300], show: true) {
    borderLayout()
    textlabel = label(text: 'Click the button!', constraints: BL.NORTH)
    button(text:'Click Me',
         actionPerformed: {count++; textlabel.text = "Clicked ${count} time(s)."; println "clicked"}, constraints:BL.SOUTH)
  }
}
Here is what it will look like:

SwingBuilder001
This hierarchy of components would normally be created through a series of repetitive instantiations, setters, and finally attaching this child to its respective parent. Using SwingBuilder, however, allows you to define this hierarchy in its native form, which makes the interface design understandable simply by reading the code.

The flexibility shown here is made possible by leveraging the many programming features built-in to Groovy, such as closures, implicit constructor calling, import aliasing, and string interpolation. Of course, these do not have to be fully understood in order to use SwingBuilder; as you can see from the code above, their uses are intuitive.

Here is a slightly more involved example, with an example of SwingBuilder code re-use via a closure.

import groovy.swing.SwingBuilder
import javax.swing.*
import java.awt.*

def swing = new SwingBuilder()

def sharedPanel = {
     swing.panel() {
        label("Shared Panel")
    }
}

count = 0
swing.edt {
    frame(title: 'Frame', defaultCloseOperation: JFrame.EXIT_ON_CLOSE, pack: true, show: true) {
        vbox {
            textlabel = label('Click the button!')
            button(
                text: 'Click Me',
                actionPerformed: {
                    count++
                    textlabel.text = "Clicked ${count} time(s)."
                    println "Clicked!"
                }
            )
            widget(sharedPanel())
            widget(sharedPanel())
        }
    }
}
Here’s another variation that relies on observable beans and binding:

import groovy.swing.SwingBuilder
import groovy.beans.Bindable

class MyModel {
   @Bindable int count = 0
}

def model = new MyModel()
new SwingBuilder().edt {
  frame(title: 'Java Frame', size: [100, 100], locationRelativeTo: null, show: true) {
    gridLayout(cols: 1, rows: 2)
    label(text: bind(source: model, sourceProperty: 'count', converter: { v ->  v? "Clicked $v times": ''}))
    button('Click me!', actionPerformed: { model.count++ })
  }
}
@Bindable is one of the core AST Transformations. It generates all the required boilerplate code to turn a simple bean into an observable one. The bind() node creates appropriate PropertyChangeListeners that will update the interested parties whenever a PropertyChangeEvent is fired.

8.2.10. AntBuilder

Despite being primarily a build tool, Apache Ant is a very practical tool for manipulating files including zip files, copy, resource processing, …​ But if ever you’ve been working with a build.xml file or some Jelly script and found yourself a little restricted by all those pointy brackets, or found it a bit weird using XML as a scripting language and wanted something a little cleaner and more straight forward, then maybe Ant scripting with Groovy might be what you’re after.

Groovy has a helper class called AntBuilder which makes the scripting of Ant tasks really easy; allowing a real scripting language to be used for programming constructs (variables, methods, loops, logical branching, classes etc). It still looks like a neat concise version of Ant’s XML without all those pointy brackets; though you can mix and match this markup inside your script. Ant itself is a collection of jar files. By adding them to your classpath, you can easily use them within Groovy as is. We believe using AntBuilder leads to more concise and readily understood syntax.

AntBuilder exposes Ant tasks directly using the convenient builder notation that we are used to in Groovy. Here is the most basic example, which is printing a message on the standard output:

def ant = new AntBuilder()          
ant.echo('hello from Ant!')         
creates an instance of AntBuilder
executes the echo task with the message in parameter
Imagine that you need to create a ZIP file:

def ant = new AntBuilder()
ant.zip(destfile: 'sources.zip', basedir: 'src')
In the next example, we demonstrate the use of AntBuilder to copy a list of files using a classical Ant pattern directly in Groovy:

// lets just call one task
ant.echo("hello")

// here is an example of a block of Ant inside GroovyMarkup
ant.sequential {
    echo("inside sequential")
    def myDir = "target/AntTest/"
    mkdir(dir: myDir)
    copy(todir: myDir) {
        fileset(dir: "src/test") {
            include(name: "**/*.groovy")
        }
    }
    echo("done")
}

// now lets do some normal Groovy again
def file = new File(ant.project.baseDir,"target/AntTest/groovy/util/AntTest.groovy")
assert file.exists()
Another example would be iterating over a list of files matching a specific pattern:

// lets create a scanner of filesets
def scanner = ant.fileScanner {
    fileset(dir:"src/test") {
        include(name:"**/Ant*.groovy")
    }
}

// now lets iterate over
def found = false
for (f in scanner) {
    println("Found file $f")
    found = true
    assert f instanceof File
    assert f.name.endsWith(".groovy")
}
assert found
Or execute a JUnit test:

// lets create a scanner of filesets
ant.junit {
    test(name:'groovy.util.SomethingThatDoesNotExist')
}
We can even go further by compiling and executing a Java file directly from Groovy:

ant.echo(file:'Temp.java', '''
    class Temp {
        public static void main(String[] args) {
            System.out.println("Hello");
        }
    }
''')
ant.javac(srcdir:'.', includes:'Temp.java', fork:'true')
ant.java(classpath:'.', classname:'Temp', fork:'true')
ant.echo('Done')
It is worth mentioning that AntBuilder is included in Gradle, so you can use it in Gradle just like you would in Groovy. Additional documentation can be found in the Gradle manual.

8.2.11. CliBuilder

(TBD)

8.2.12. ObjectGraphBuilder

ObjectGraphBuilder is a builder for an arbitrary graph of beans that follow the JavaBean convention. It is in particular useful for creating test data.

Let’s start with a list of classes that belong to your domain:

package com.acme

class Company {
    String name
    Address address
    List employees = []
}

class Address {
    String line1
    String line2
    int zip
    String state
}

class Employee {
    String name
    int employeeId
    Address address
    Company company
}
Then using ObjectGraphBuilder building a Company with three employees is as easy as:

def builder = new ObjectGraphBuilder()                          
builder.classLoader = this.class.classLoader                    
builder.classNameResolver = "com.acme"                          

def acme = builder.company(name: 'ACME') {                      
    3.times {
        employee(id: it.toString(), name: "Drone $it") {        
            address(line1:"Post street")                        
        }
    }
}

assert acme != null
assert acme instanceof Company
assert acme.name == 'ACME'
assert acme.employees.size() == 3
def employee = acme.employees[0]
assert employee instanceof Employee
assert employee.name == 'Drone 0'
assert employee.address instanceof Address
creates a new object graph builder
sets the classloader where the classes will be resolved
sets the base package name for classes to be resolved
creates a Company instance
with 3 Employee instances
each of them having a distinct Address
Behind the scenes, the object graph builder:

will try to match a node name into a Class, using a default ClassNameResolver strategy that requires a package name

then will create an instance of the appropriate class using a default NewInstanceResolver strategy that calls a no-arg constructor

resolves the parent/child relationship for nested nodes, involving two other strategies:

RelationNameResolver will yield the name of the child property in the parent, and the name of the parent property in the child (if any, in this case, Employee has a parent property aptly named company)

ChildPropertySetter will insert the child into the parent taking into account if the child belongs to a Collection or not (in this case employees should be a list of Employee instances in Company).

All 4 strategies have a default implementation that work as expected if the code follows the usual conventions for writing JavaBeans. In case any of your beans or objects do not follow the convention you may plug your own implementation of each strategy. For example imagine that you need to build a class which is immutable:

@Immutable
class Person {
    String name
    int age
}
Then if you try to create a Person with the builder:

def person = builder.person(name:'Jon', age:17)
It will fail at runtime with:

Cannot set readonly property: name for class: com.acme.Person
Fixing this can be done by changing the new instance strategy:

builder.newInstanceResolver = { Class klazz, Map attributes ->
    if (klazz.isAnnotationPresent(Immutable)) {
        def o = klazz.newInstance(attributes)
        attributes.clear()
        return o
    }
    klazz.newInstance()
}
ObjectGraphBuilder supports ids per node, meaning that you can store a reference to a node in the builder. This is useful when multiple objects reference the same instance. Because a property named id may be of business meaning in some domain models ObjectGraphBuilder has a strategy named IdentifierResolver that you may configure to change the default name value. The same may happen with the property used for referencing a previously saved instance, a strategy named ReferenceResolver will yield the appropriate value (default is `refId'):

def company = builder.company(name: 'ACME') {
    address(id: 'a1', line1: '123 Groovy Rd', zip: 12345, state: 'JV')          
    employee(name: 'Duke', employeeId: 1, address: a1)                          
    employee(name: 'John', employeeId: 2 ){
      address( refId: 'a1' )                                                    
    }
}
an address can be created with an id
an employee can reference the address directly with its id
or use the refId attribute corresponding to the id of the corresponding address
Its worth mentioning that you cannot modify the properties of a referenced bean.

8.2.13. JmxBuilder

See Working with JMX - JmxBuilder for details.

8.2.14. FileTreeBuilder

FileTreeBuilder is a builder for generating a file directory structure from a specification. For example, to create the following tree:

.. code-block:: shell

 src/
  |--- main
  |     |--- groovy
  |            |--- Foo.groovy
  |--- test
        |--- groovy
               |--- FooTest.groovy


You can use a FileTreeBuilder like this:

tmpDir = File.createTempDir()
def fileTreeBuilder = new FileTreeBuilder(tmpDir)
fileTreeBuilder.dir('src') {
    dir('main') {
       dir('groovy') {
          file('Foo.groovy', 'println "Hello"')
       }
    }
    dir('test') {
       dir('groovy') {
          file('FooTest.groovy', 'class FooTest extends GroovyTestCase {}')
       }
    }
 }
To check that everything worked as expected we use the following `assert`s:

assert new File(tmpDir, '/src/main/groovy/Foo.groovy').text == 'println "Hello"'
assert new File(tmpDir, '/src/test/groovy/FooTest.groovy').text == 'class FooTest extends GroovyTestCase {}'
FileTreeBuilder also supports a shorthand syntax:

tmpDir = File.createTempDir()
def fileTreeBuilder = new FileTreeBuilder(tmpDir)
fileTreeBuilder.src {
    main {
       groovy {
          'Foo.groovy'('println "Hello"')
       }
    }
    test {
       groovy {
          'FooTest.groovy'('class FooTest extends GroovyTestCase {}')
       }
    }
 }
This produces the same directory structure as above, as shown by these `assert`s:

assert new File(tmpDir, '/src/main/groovy/Foo.groovy').text == 'println "Hello"'
assert new File(tmpDir, '/src/test/groovy/FooTest.groovy').text == 'class FooTest extends GroovyTestCase {}'