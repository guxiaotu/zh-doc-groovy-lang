运行时&编译时元编程
=====================


Groovy 提供两类元编程，分别为:运行时元编程与编译时元编程。第一种允许在运行时改变类模型，而第二种发生在编译时。其各有优劣之处，这一章节我们详细讲解。

运行时元编程
-------------------------


在运行时元编程中，我们在运行时拦截，注入甚至合成类和接口的方法。为了深入理解 Groovy MOP ， 我们需要理解 Groovy 的对象及方法的处理方式。Groovy 中有三类对象：POJO，POGO 和 Groovy 拦截器。Groovy 中对于以上对象均能使用元编程，但使用方式会有差别。

- ``POJO`` - 正规的 Java 对象，其类可以用Java或任何其他 JVM 语言编写。
- ``POGO`` - Groovy 对象，其类使用 Groovy 编写。继承于 ``java.lang.Object`` 并且实现 `groovy.lang.GroovyObject <http://docs.groovy-lang.org/2.4.5/html/gapi/index.html?groovy/lang/GroovyObject.html>`_ 接口。
- ``Groovy Interceptor`` - Groovy 对象，实现 ``groovy.lang.GroovyInterceptable`` 接口，具有方法拦截能力，我们将在 ``GroovyInterceptable`` 章节详细讲解。

For every method call Groovy checks whether the object is a POJO or a POGO. For POJOs, Groovy fetches it’s MetaClass from the groovy.lang.MetaClassRegistry and delegates method invocation to it. For POGOs, Groovy takes more steps, as illustrated in the following figure:

每个方法调用过程中， Groovy 检查当前对象是 POJO 还是 POGO。对于 POJOs ， Groovy 将从 ``groovy.lang.MetaClassRegistry`` 中取出其 ``MetaClass`` 并使用代理方式进行方法调用。对于 POGOs，Groovy 将执行更多步骤，如下图


.. image:: images/GroovyInterceptions.png
    :alt: Groovy interception mechanism
    :align: center

GroovyObject interface
^^^^^^^^^^^^^^^^^^^^^^

``groovy.lang.GroovyObject`` 作为 Groovy 中的主要接口，就像 Java 中的 ``Object`` 类。
``GroovyObject``  的在 ``groovy.lang.GroovyObjectSupport`` 中默认实现,  其主要负责将调用传递给 ``MetaClass`` 对象。
参考下面代码：
 
 .. code-block:: groovy
 
    package groovy.lang;

    public interface GroovyObject {

        Object invokeMethod(String name, Object args);

        Object getProperty(String propertyName);

        void setProperty(String propertyName, Object newValue);

        MetaClass getMetaClass();

        void setMetaClass(MetaClass metaClass);
    }

invokeMethod
"""""""""""""""""""""""

在运行时元编程模式下，当 Groovy 对象上调用的方法不存在时，将会调用  ``invokeMethod`` 方法。
下面的例子中，就是用到重载 ``invokeMethod`` 方法：

.. code-block:: groovy

    class SomeGroovyClass {

        def invokeMethod(String name, Object args) {
            return "called invokeMethod $name $args"
        }

        def test() {
            return 'method exists'
        }
    }

    def someGroovyClass = new SomeGroovyClass()

    assert someGroovyClass.test() == 'method exists'
    assert someGroovyClass.someMethod() == 'called invokeMethod someMethod []'



get/getProperty
""""""""""""""""""""""""

对象上读取属性操作都将通过重载 getProperty 方法拦截，例如：

.. code-block:: groovy

    class SomeGroovyClass {

        def property1 = 'ha'
        def field2 = 'ho'
        def field4 = 'hu'

        def getField1() {
            return 'getHa'
        }

        def getProperty(String name) {
            if (name != 'field3')
                return metaClass.getProperty(this, name)                    // <1>
            else
                return 'field3'
        }
    }

    def someGroovyClass = new SomeGroovyClass()

    assert someGroovyClass.field1 == 'getHa'
    assert someGroovyClass.field2 == 'ho'
    assert someGroovyClass.field3 == 'field3'
    assert someGroovyClass.field4 == 'hu'
    
<1> Forwards the request to the getter for all properties except field3.

通过重载 setProperty  方法可以拦截属性修改操作：

.. code-block:: groovy

    class POGO {

        String property

        void setProperty(String name, Object value) {
            this.@"$name" = 'overriden'
        }
    }

    def pogo = new POGO()
    pogo.property = 'a'

    assert pogo.property == 'overriden'


get/setMetaClass
""""""""""""""""""""""""""""

You can a access a objects metaClass or set your own MetaClass implementation for changing the default interception mechanism. For example you can write your own implementation of the MetaClass interface and assign to it to objects and accordingly change the interception mechanism:

通过访问对象的 metaClass 或 重新设置你自己的 MetaClass 实现，可以改变默认的拦截机制。
下面例子中，你可以通过自己实现一个 MetaClass 并赋值给其对象上，这样就可以改变拦截机制：

.. code-block:: groovy

    // getMetaclass
    someObject.metaClass

    // setMetaClass
    someObject.metaClass = new OwnMetaClassImplementation()

在 ``GroovyInterceptable`` 章节中有更多实例用于参考。

get/setAttribute
^^^^^^^^^^^^^^^^

This functionality is related to the MetaClass implementation. In the default implementation you can access fields without invoking their getters and setters. The examples below demonstrate this approach:

这一功能与 MetaClass 的具体实现有关。在默认的实现中，你可以无需调用 fields 上的 getters  和 setters 方法，就可以通过 ``get/setAttribute`` 访问控制 fields 。
下面例子中将展示这种特性：

.. code-block:: groovy

    class SomeGroovyClass {

        def field1 = 'ha'
        def field2 = 'ho'

        def getField1() {
            return 'getHa'
        }
    }

    def someGroovyClass = new SomeGroovyClass()

    assert someGroovyClass.metaClass.getAttribute(someGroovyClass, 'field1') == 'ha'
    assert someGroovyClass.metaClass.getAttribute(someGroovyClass, 'field2') == 'ho'


.. code-block:: groovy

    class POGO {

        private String field
        String property1

        void setProperty1(String property1) {
            this.property1 = "setProperty1"
        }
    }

    def pogo = new POGO()
    pogo.metaClass.setAttribute(pogo, 'field', 'ha')
    pogo.metaClass.setAttribute(pogo, 'property1', 'ho')

    assert pogo.field == 'ha'
    assert pogo.property1 == 'ho'


methodMissing
^^^^^^^^^^^^^^^^^^

Groovy 支持 ``methodMissing`` 概念. 当方法调用失败，方法的名称或参数不正确，都将调用 ``methodMissing`` 方法。

.. code-block:: groovy

    class Foo {

       def methodMissing(String name, def args) {
            return "this is me"
       }
    }

    assert new Foo().someUnknownMethod(42l) == 'this is me'


通常情况下，使用 ``methodMissing`` ，其可以缓存相同方法及参数调用结果，并提供给下次调用使用。

For example consider dynamic finders in GORM. These are implemented in terms of methodMissing. The code resembles something like this:
例如在 GORM 中的动态查找，这里就实现了 methodMissing 。可以参考类似代码：

.. code-block:: groovy

    class GORM {

       def dynamicMethods = [...] // an array of dynamic methods that use regex

       def methodMissing(String name, args) {
           def method = dynamicMethods.find { it.match(name) }
           if(method) {
              GORM.metaClass."$name" = { Object[] varArgs ->
                 method.invoke(delegate, name, varArgs)
              }
              return method.invoke(delegate,name, args)
           }
           else throw new MissingMethodException(name, delegate, args)
       }
    }



这里需要注意，如果我们找到调用的方法，然后将其注册到 ``ExpandoMetaClass`` , 在下次再次调用此方法，将会更加高效。
这是使用 ``methodMissing`` 不会有调用 ``invokeMethod`` 的开销，在第二次调用此方法，就和原生方法一样。

propertyMissing
^^^^^^^^^^^^^^^

当访问的属性不存在时，会调用 ``propertyMissing``。
``propertyMissing``  方法只有一个字符串类型的入参，其参数为调用的属性名称。

.. code-block:: groovy

    class Foo {
       def propertyMissing(String name) { name }
    }

    assert new Foo().boo == 'boo'

在运行时，当无法找到属性对应的 getter 方法的情况下，才能调用 ``propertyMissing(String)`` 方法。


对于 setter 方法， 第二个 ``propertyMissing`` 方法定义增加了一个 ``value`` 参数：

.. code-block:: groovy

    class Foo {
       def storage = [:]
       def propertyMissing(String name, value) { storage[name] = value }
       def propertyMissing(String name) { storage[name] }
    }

    def f = new Foo()
    f.foo = "bar"

    assert f.foo == "bar"


相比较于 ``methodMissing`` , 这是最好方式在运行时动态注册属性并能提升整体的查找性能。

``methodMissing`` 和 ``propertyMissing``  处理的方法和属性，都可以通过 ``ExpandoMetaClass`` 添加注册。


GroovyInterceptable
^^^^^^^^^^^^^^^^^^^

``groovy.lang.GroovyInterceptable`` 是继承于 ``GroovyObject``  的关键接口，实现此接口的类上的方法调用都将经过运行
时方法路由机制拦截。

.. code-block:: groovy

    package groovy.lang;

    public interface GroovyInterceptable extends GroovyObject {
    }

当 Groovy 对象实现 ``GroovyInterceptable`` 接口，其对象上的任何方法调用都将通过 ``invokeMethod()``  拦截器。
可以参考下面的例子：

.. code-block:: groovy

    class Interception implements GroovyInterceptable {

        def definedMethod() { }

        def invokeMethod(String name, Object args) {
            'invokedMethod'
        }
    }

下面代码中会看到，无论被调用方式是否存在，返回值都是相同的：

.. code-block:: groovy

    class InterceptableTest extends GroovyTestCase {

        void testCheckInterception() {
            def interception = new Interception()

            assert interception.definedMethod() == 'invokedMethod'
            assert interception.someMethod() == 'invokedMethod'
        }
    }



需要注意，我们不能使用像 ``println``  这样的方法，是由于其已经被注入到所有 groovy 对象中，也将会被拦截。

If we want to intercept all methods call but do not want to implement the GroovyInterceptable interface we can implement invokeMethod() on an object’s MetaClass. This approach works for both POGOs and POJOs, as shown by this example:
如果你想拦截所有方法，但是有不想实现 ``GroovyInterceptable`` 接口，可以通过在对象的 ``MetaClass`` 方法上实现 ``invokeMethod`` 方法来达到同样的效果。
这种方法对于 ``POGOs`` 和 ``POJOs``   都适用，看看下面的例子：

.. code-block:: groovy

    class InterceptionThroughMetaClassTest extends GroovyTestCase {

        void testPOJOMetaClassInterception() {
            String invoking = 'ha'
            invoking.metaClass.invokeMethod = { String name, Object args ->
                'invoked'
            }

            assert invoking.length() == 'invoked'
            assert invoking.someMethod() == 'invoked'
        }

        void testPOGOMetaClassInterception() {
            Entity entity = new Entity('Hello')
            entity.metaClass.invokeMethod = { String name, Object args ->
                'invoked'
            }

            assert entity.build(new Object()) == 'invoked'
            assert entity.someMethod() == 'invoked'
        }
    }

更多有关 MetaClass 的内容可以查看对应`章节 <http://www.groovy-lang.org/metaprogramming.html#_metaclasses>`_ 。

Categories
^^^^^^^^^^^^^^^^

当我们对于无法控制的类，需要增加附加方法时，分类这种方式非常有用。
对于这种特性，``Groovy``  的实现是借鉴于 ``Objective-C`` ，并称其为分类。

分类通过称为 ``category`` 类来实现。``category`` 类中需要根据预定规则来定义扩展方法。

这里有一些分类用于扩展相应类的功能，这种方式可以使其在 Groovy 中发挥更强的作用：

- `groovy.time.TimeCategory <http://docs.groovy-lang.org/2.4.5/html/gapi/index.html?groovy/time/TimeCategory.html>`_

- `groovy.servlet.ServletCategory <http://docs.groovy-lang.org/2.4.5/html/gapi/index.html?groovy/servlet/ServletCategory.html>`_

- `groovy.xml.dom.DOMCategory <http://docs.groovy-lang.org/2.4.5/html/gapi/index.html?groovy/xml/dom/DOMCategory.html>`_
  

``Category`` 类在默认情况下并不开启。当使用 ``category``  中定义的方法时，需要使用 GDK 中提供的 ``use``  方法。

.. code-block:: groovy

    use(TimeCategory)  {
        println 1.minute.from.now           // <1>
        println 10.hours.ago

        def someDate = new Date()         // <2>
        println someDate - 3.months
    }

<1> TimeCategory adds methods to Integer
<2> TimeCategory adds methods to Date

``use`` 方法第一个参数为具体定义 ``category`` 类，第二个参数为闭包代码块。
在闭包块中可以访问 ``category`` 类中的方法。
正如上面例子中看到的， JDK 中 ``java.lang.Integer`` 或 ``java.util.Date``  类也可以通过分类中定义的方法来增强。

分类也可以进行一定的封装，像下面这样：

.. code-block:: groovy

    class JPACategory{
      // Let's enhance JPA EntityManager without getting into the JSR committee
      static void persistAll(EntityManager em , Object[] entities) { //add an interface to save all
        entities?.each { em.persist(it) }
      }
    }

    def transactionContext = {
      EntityManager em, Closure c ->
      def tx = em.transaction
      try {
        tx.begin()
        use(JPACategory) {
          c()
        }
        tx.commit()
      } catch (e) {
        tx.rollback()
      } finally {
        //cleanup your resource here
      }
    }

    // user code, they always forget to close resource in exception, some even forget to commit, let's not rely on them.
    EntityManager em; //probably injected
    transactionContext (em) {
     em.persistAll(obj1, obj2, obj3)
     // let's do some logics here to make the example sensible
     em.persistAll(obj2, obj4, obj6)
    }


当我们看到  ``groovy.time.TimeCategory``  中所定义的方法都声明为 static 。
这种声明方式是通过  ``category``  方式在 use 代码块中调用扩展方法的必要条件。

.. code-block:: groovy

    public class TimeCategory {

    public static Date plus(final Date date, final BaseDuration duration) {
        return duration.plus(date);
    }

    public static Date minus(final Date date, final BaseDuration duration) {
        final Calendar cal = Calendar.getInstance();

        cal.setTime(date);
        cal.add(Calendar.YEAR, -duration.getYears());
        cal.add(Calendar.MONTH, -duration.getMonths());
        cal.add(Calendar.DAY_OF_YEAR, -duration.getDays());
        cal.add(Calendar.HOUR_OF_DAY, -duration.getHours());
        cal.add(Calendar.MINUTE, -duration.getMinutes());
        cal.add(Calendar.SECOND, -duration.getSeconds());
        cal.add(Calendar.MILLISECOND, -duration.getMillis());

        return cal.getTime();
    }

    // ...

另一个必要条件是静态方法中的第一个参数类型需要与将要使用的类型一致。
其他参数为方法调用中的正常参数。

由于参数及静态方法的约定，分类中方法的定义与普通方法比较起来，显得不太直观。
 Groovy 可以通过 ``@Category`` 注解的方式，将经过注解的类在编译期转化为 ``category`` 类型。

.. code-block:: groovy

    class Distance {
        def number
        String toString() { "${number}m" }
    }

    @Category(Number)
    class NumberCategory {
        Distance getMeters() {
            new Distance(number: this)
        }
    }

    use (NumberCategory)  {
        assert 42.meters.toString() == '42m'
    }


使用 ``@Category`` 的优势是在使用实例方法时，可以不需要将第一个参数声明为目标类型。
其目标类型的设置通过注解方式替代。

这里 `编译时元编程 <http://www.groovy-lang.org/metaprogramming.html#xform-Category>`_  章节将介绍 ``@Category`` 的其他用法。

Metaclasses
^^^^^^^^^^^^^^^^^^
待续
(TBD)

Custom metaclasses
""""""""""""""""""""""""""""

待续
(TBD)

Delegating metaClass
++++++++++++++++++++

待续

Magic package(Maksym Stavytskyi)
++++++++++++++++++++++++++++++++++++++++++++

待续
(TBD)

Per instance metaclass
""""""""""""""""""""""""""""""""""""""

待续
(TBD)

ExpandoMetaClass
"""""""""""""""""""""""""""""

``Groovy`` 中自带特殊的 ``MetaClass``  称为 ``ExpandoMetaClass`` 。
特殊之处就在于，可以通过使用闭包方式动态的添加或改变方法，构造器，属性以及静态方法。

这些动态的处理，对于在 `Test guide <http://docs.groovy-lang.org/latest/html/documentation/core-testing-guide.html#testing_guide_emc>`_  章节中看到的 ``moking`` 和 ``stubbing`` 应用场景非常有用。

Groovy 对每个 ``java.lang.Class`` 都提供了一个 ``metaClass`` 特殊属性，其返回一个 ``ExpandoMetaClass`` 实例的引用。
在这实例上可以添加方法或修改已存在的方法内容。


接下来介绍 ``ExpandoMetaClass``  在各种场景中的使用。


Methods
++++++++++++


通过调用 ``metaClass`` 属性来获取 ``ExpandoMetaClass`` ，使用 ``<<`` 或 ``=``  操作符添加方法。
Note that the left shift operator is used to append a new method. 
注意 ``<<``  用于添加新方法，如果这个方法已经存在，将会抛出异常。如果你想替换方法，可以使用 ``=`` 

这些操作符向 metaClass 的不存在的属性传递 闭包代码块的实例。

.. code-block:: groovy

    class Book {
       String title
    }

    Book.metaClass.titleInUpperCase << {-> title.toUpperCase() }

    def b = new Book(title:"The Stand")

    assert "THE STAND" == b.titleInUpperCase()


上面例子中就是通过通过访问 ``metaClass`` 并使用 ``<<`` 或 ``=`` 操作符, 赋值闭包代码块的方式向类中添加新增方法。
无参数方法可以使用 ``{-> ...}`` 方式添加。


Properties
++++++++++++++

``ExpandoMetaClass`` 支持两种方式添加，重载属性。

第一种，在 ``metaClass`` 上直接声明一个属性并赋值：

.. code-block:: groovy

    class Book {
       String title
    }

    Book.metaClass.author = "Stephen King"
    def b = new Book()

    assert "Stephen King" == b.author

另一种方式，添加属性对应的 getter 或 setter 方法：

.. code-block:: groovy

    class Book {
     String title
    }
    Book.metaClass.getAuthor << {-> "Stephen King" }

    def b = new Book()

    assert "Stephen King" == b.author


上面代码中属性通过闭包指定，并且为只读。
这里可以添加 setter 方法来存储属性值，以便后续使用。可以看看下面代码：

.. code-block:: groovy

    class Book {
      String title
    }

    def properties = Collections.synchronizedMap([:])

    Book.metaClass.setAuthor = { String value ->
       properties[System.identityHashCode(delegate) + "author"] = value
    }
    Book.metaClass.getAuthor = {->
       properties[System.identityHashCode(delegate) + "author"]
    }

例如在 Servlet 容器中将当前执行的 request 中的值存储到 request  attributes 中。
Grails 中也有相同的应用。

Constructors
++++++++++++

构造器添加使用特殊属性 ``constructor`` 。
同样适用 ``<<`` 或 ``=`` 来传递闭包代码块。在代码执行时，闭包中定义的参数列表就为构造器的参数列表。

.. code-block:: groovy

    class Book {
        String title
    }
    Book.metaClass.constructor << { String title -> new Book(title:title) }

    def book = new Book('Groovy in Action - 2nd Edition')
    assert book.title == 'Groovy in Action - 2nd Edition'

需要当心的是在添加构造器时，会比较容易出现栈溢出的问题。


Static Methods
++++++++++++++


静态方法的加入也使用同样的技术，但需要在方法名称之前添加 ``static`` 限定符。

.. code-block:: groovy

    class Book {
       String title
    }

    Book.metaClass.static.create << { String title -> new Book(title:title) }

    def b = Book.create("The Stand")


Borrowing Methods
+++++++++++++++++

在 ``ExpandoMetaClass`` 中可以使用方法指针语法，向其他类中借取方法。

.. code-block:: groovy

    class Person {
        String name
    }
    class MortgageLender {
       def borrowMoney() {
          "buy house"
       }
    }

    def lender = new MortgageLender()

    Person.metaClass.buyHouse = lender.&borrowMoney

    def p = new Person()

    assert "buy house" == p.buyHouse()


Dynamic Method Names
++++++++++++++++++++

Groovy 中可以使用 String 作为属性名称，同样可以允许你在运行时动态的创建方法或属性名称。
使用动态命名方法，直接使用了语言中引用属性名称作为字符串的特性。

.. code-block:: groovy

    class Person {
       String name = "Fred"
    }

    def methodName = "Bob"

    Person.metaClass."changeNameTo${methodName}" = {-> delegate.name = "Bob" }

    def p = new Person()

    assert "Fred" == p.name

    p.changeNameToBob()

    assert "Bob" == p.name

静态方法及属性同样适用这种概念。

Grails 中可以发现动态方法命名的应用。
动态编码器这一概念，也是通过使用动态方法命名来实现。

*HTMLCodec Class*

.. code-block:: groovy

    class HTMLCodec {
        static encode = { theTarget ->
            HtmlUtils.htmlEscape(theTarget.toString())
        }

        static decode = { theTarget ->
            HtmlUtils.htmlUnescape(theTarget.toString())
        }
    }

上面例子是一个编码器的实现。
Grails 中各种编码器都定义在各自独立的类中。
在运行环境的 ``classpath`` 中将有多个编译器类。
应用启动时将  ``encodeXXX`` 和 ``decodeXXX`` 方法添加到特定的 meta-classes 中，其中 ``XXX`` 就是编码器类名
的第一部分（例如： encodeHTML）。
这种处理机制将在下面的伪代码中展示：

 .. code-block:: groovy
 
     def codecs = classes.findAll { it.name.endsWith('Codec') }

    codecs.each { codec ->
        Object.metaClass."encodeAs${codec.name-'Codec'}" = { codec.newInstance().encode(delegate) }
        Object.metaClass."decodeFrom${codec.name-'Codec'}" = { codec.newInstance().decode(delegate) }
    }


    def html = '<html><body>hello</body></html>'

    assert '<html><body>hello</body></html>' == html.encodeAsHTML()


Runtime Discovery
+++++++++++++++++

在方法执行时，了解存在的其他方法或属性是十分有用的。 当前版本中，``ExpandoMetaClass`` 提供一下方法用于此法：

- getMetaMethod
- hasMetaMethod
- getMetaProperty
- hasMetaProperty
  
为什么不使用放射？
Groovy 中的方法分为原生方法以及运行时动态方法。
这里它们通常表示为元方法（MetaMethods）。元方法可以告诉你在运行时有效的方法。

当在重载 ``invokeMethod`` , ``getProperty`` 或 ``setProperty`` 时这将会非常有用。

GroovyObject Methods
++++++++++++++++++++

``ExpandoMetaClass`` 的另一个特性是其允许重载 ``invokeMethod`` , ``getProperty`` 和 ``setProperty`` 方法，这些方法都可以在 ``groovy.lang.GroovyObject`` 中
找到。

这里将展示如何重载 ``invokeMethod`` 方法：

.. code-block:: groovy


    class Stuff {
       def invokeMe() { "foo" }
    }

    Stuff.metaClass.invokeMethod = { String name, args ->
       def metaMethod = Stuff.metaClass.getMetaMethod(name, args)
       def result
       if(metaMethod) result = metaMethod.invoke(delegate,args)
       else {
          result = "bar"
       }
       result
    }

    def stf = new Stuff()

    assert "foo" == stf.invokeMe()
    assert "bar" == stf.doStuff()


闭包代码中的第一部分用于根据给定的方法名称及参数查找对应方法。
如果方法找到，就可以对方法进行代理处理。如果没有找到，方法返回默认值。

元方法是存在于 MetaClass 上的方法，无论是在运行时，还是编译时添加。
对于 ``setProperty``  和 ``getProperty`` 同样适用。

.. code-block:: groovy

    class Person {
       String name = "Fred"
    }

    Person.metaClass.getProperty = { String name ->
       def metaProperty = Person.metaClass.getMetaProperty(name)
       def result
       if(metaProperty) result = metaProperty.getProperty(delegate)
       else {
          result = "Flintstone"
       }
       result
    }

    def p = new Person()

    assert "Fred" == p.name
    assert "Flintstone" == p.other



The important thing to note here is that instead of a MetaMethod a MetaProperty instance is looked up. 
If that exists the getProperty method of the MetaProperty is called, passing the delegate.


Overriding Static invokeMethod
++++++++++++++++++++++++++++++


``ExpandoMetaClass`` 通过特殊的 ``invokeMethod``  语法来重载静态方法。

.. code-block:: groovy

    class Stuff {
       static invokeMe() { "foo" }
    }

    Stuff.metaClass.'static'.invokeMethod = { String name, args ->
       def metaMethod = Stuff.metaClass.getStaticMetaMethod(name, args)
       def result
       if(metaMethod) result = metaMethod.invoke(delegate,args)
       else {
          result = "bar"
       }
       result
    }

    assert "foo" == Stuff.invokeMe()
    assert "bar" == Stuff.doStuff()

这里重载逻辑与前面看的重载实例方法是一样的。
唯一的区别是使用 ``metaClass.static`` 访问属性，调用 ``getStaticMethodName`` 来获取静态元方法实例。


Extending Interfaces
++++++++++++++++++++

使用 ``ExpandoMetaClass`` 可以在接口上添加方法。
要做到这一点，在应用启动时，需要在全局范围调用 ``ExpandoMetaClass.enableGlobally()`` 方法。

.. code-block:: groovy

    List.metaClass.sizeDoubled = {-> delegate.size() * 2 }

    def list = []

    list << 1
    list << 2

    assert 4 == list.sizeDoubled()


Extension modules
^^^^^^^^^^^^^^^^^

Extending existing classes
""""""""""""""""""""""""""

An extension module allows you to add new methods to existing classes, including classes which are precompiled, like classes from the JDK. Those new methods, unlike those defined through a metaclass or using a category, are available globally. For example, when you write:

扩展模块允许你在已有的类上添加新的方法，包括已经预编译的类，如 JDK 中的类。
这里新添加的方法，不同于通过 metaclass 或 category 定义的方法，其可以在全局使用。 例如：

*Standard extension method*

.. code-block:: groovy

    def file = new File(...)
    def contents = file.getText('utf-8')


``getText`` 方法并不存在于 ``File`` 类中。然而在 Groovy 在能使用，是由于其定义在一个称为 ``ResourceGroovyMethods`` 特殊类中：

*ResourceGroovyMethods.java*

.. code-block:: groovy

    public static String getText(File file, String charset) throws IOException {
     return IOGroovyMethods.getText(newReader(file, charset));
    }


你需要注意的是，这个扩展方法使用 ``static`` 定义在帮助类中（其他扩展方法都在这里定义）。
``getText`` 第一个参数对应其接受对象，其他参数对应这个扩展方法上的参数。
这样就可以在 ``File`` 类上调用 ``getText`` 方法。

创建一个扩展模块过程非常简单：

- 想上面编写一个扩展方法
- 编写一个模块描述文件

然后你需要将扩展模块及描述文件配置到 ``classpath`` 中，这样在 Groovy 中就可以使用。
这意味着，你可以选择：

- 直接将类与模块描述文件配置在 ``classpath`` 中
- 或将其打包至 ``jar`` 中，方便能够重用

扩展模块可以添加两类方法：

- 实例方法（类实例上调用的方法）
- 静态方法（类自身调用方法）

Instance methods
""""""""""""""""

在类上添加实例方法，需要创建新的扩展类。
例如，你想在 ``Integer`` 上添加一个 ``maxRetries`` 方法，其方法接受一个闭包，在没有异常发生的情况下可以执行 `n` 次。
想实现上面的需求，你可以这样：

*MaxRetriesExtension.groovy*

.. code-block:: groovy

    class MaxRetriesExtension {                                       // <1>                     
        static void maxRetries(Integer self, Closure code) {          // <2>
            int retries = 0
            Throwable e
            while (retries<self) {
                try {
                    code.call()
                    break
                } catch (Throwable err) {
                    e = err
                    retries++
                }
            }
            if (retries==0 && e) {
                throw e
            }
        }
    }


<1> The extension class
<2> First argument of the static method corresponds to the receiver of the message, that is to say the extended instance

在 `声明 <http://www.groovy-lang.org/metaprogramming.html#module-descriptor>`_ 你的扩展类之后，你可以这样调用：

.. code-block:: groovy

    int i=0
    5.maxRetries {
        i++
    }
    assert i == 1
    i=0
    try {
        5.maxRetries {
            throw new RuntimeException("oops")
        }
    } catch (RuntimeException e) {
        assert i == 5
    }



Static methods
""""""""""""""

向类中也可以添加静态方法。
在这种情况下，静态方法需要定义在独立的文件中。
静态扩展方法和实例扩展方法不能在同一个类中定义。

*StaticStringExtension.groovy*

.. code-block:: groovy

    class StaticStringExtension {                               // <1>                              
        static String greeting(String self) {                   // <2>        
            'Hello, world!'
        }
    }


<1> The static extension class
<2> First argument of the static method corresponds to the class being extended and is unused

这样你可以直接在 String 类上调用：

.. code-block:: groovy

    assert String.greeting() == 'Hello, world!'


模块描述文件 （Module descriptor）
"""""""""""""""""""""""""""""""""""""""""""

在 Groovy 中加载扩展方法，需要声明其扩展类。
需要在 ``META-INF／services`` 目录下创建 ``org.codehaus.groovy.runtime.ExtensionModule`` 文件。

*org.codehaus.groovy.runtime.ExtensionModule*

.. code-block:: groovy

    moduleName=Test module for specifications
    moduleVersion=1.0-test
    extensionClasses=support.MaxRetriesExtension
    staticExtensionClasses=support.StaticStringExtension


描述文件中需要 4 个属性：

- moduleName : 模块名称
- moduleVersion: 模块版本号。需要注意，版本号用于检查你不会加载相同模块的不同版本。
- extensionClasses: 扩展类列表。你可以填写多个类，使用逗号分割。
- staticExtensionClasses: 静态方法扩展类列表。你可以填写多个类，使用逗号分割。

请注意，并不要求在一个模块上同时定义静态方法与实例方法，你可以在一个模块中添加多个类。
你也可以在一个模块中扩展多个不同的类。甚至可以在一个扩展类中，对多个类型进行扩展，但是建议能针对扩展方法进行分类。

Extension modules and classpath
"""""""""""""""""""""""""""""""

It’s worth noting that you can’t use an extension which is compiled at the same time as code using it.
需要注意的是，在编译后直接通过代码调用是无法使用此扩展的。
这意味着使用扩展，需要在代码调用其之前，将其编译后的 classes 配置的到 ``classpath``。


类型检查兼容性 （Compatibility with type checking）
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

与非类不同，扩展模块适用于类型检查：如果其配置在 ``classpath`` 中，类型检查器可以检查到扩展方法。也同样适用于静态编译。



编译时元编程 （Compile-time metaprogramming）
---------------------------------------------


Groovy 中的编译时元编程允许在编译时生成代码。
这种转换是在程序的修改抽象语法树（AST），在 Groovy 中我们称其为抽象语法树转换（AST transformations）。
抽象语法树转换允许你在编译过程中修改 AST 并继续执行编译过程来生成字节码。
相比较于运行时元编程，其优点在于，任何变化在 class 文件中都是可见的（字节码中可见）。
在字节码中可见是很重要的，例如，如果你变化类的一部分（实现接口，扩展抽象类，等等）或者你需要 Java 来调用你的类。
抽象语法树转换可以在 class 中添加方法。在使用运行时元编程中，新方法只能在 Groovy 被发现调用。
如果相同方法使用编译时元编程，这个方法在 Java 中同样可见。
不仅如此，编译时元编程也带来了更好的性能表现（这个过程中无需初始化阶段）。

这一章节，我们开始讲解当前版本 Groovy 中的各种编译时转换。
在随后的章节中，我们也会介绍如何 `实现抽象语法树 <http://www.groovy-lang.org/metaprogramming.html#developing-ast-xforms>`_ 以及这种技术的缺点。

Available AST transformations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

抽象语法树转换分为两种类别：

- global AST transformations are applied transparently, globally, as soon as they are found on compile classpath
- local AST transformations are applied by annotating the source code with markers. Unlike global AST transformations, local AST transformations may support parameters.

Groovy doesn’t ship with any global AST transformation, but you can find a list of local AST transformations available for you to use in your code here:


代码生成转换（Code generation transformations）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

这种类别转换包括抽象语法树转换，能帮助去除代码样板。
这些代码都是些你必须写，但又不附带任何有用信息。
通过自动化生成代码模版，接下来需要写的代码将更加整洁，这样引入错误的可能性也会降低。


@groovy.transform.ToString
++++++++++++++++++++++++++

``@ToString`` 抽象语法树转换，生成可读性更强 ``toString`` 方法。
例如，在 Person 上进行注解，将在其上自动生成 toString 方法：

.. code-block:: groovy

    import groovy.transform.ToString

    @ToString
    class Person {
        String firstName
        String lastName
    }


这样定义，通过下面的断言测试，意味着生成的 toString 方法将属性值打印出：

.. code-block:: groovy

    def p = new Person(firstName: 'Jack', lastName: 'Nicholson')
    assert p.toString() == 'Person(Jack, Nicholson)'

下面列表中总结了 @ToString 可以接收的参数：

(TBD)


@groovy.transform.EqualsAndHashCode
+++++++++++++++++++++++++++++++++++

The @EqualsAndHashCode AST transformation aims at generating equals and hashCode methods for you. 
``@EqualsAndHashCode`` 用于生成 ``equals`` 和 ``hashcode`` 方法。
生成 ``hashcode`` 方法依照 ``Effective Java by Josh Bloch`` 中描述的做法：

.. code-block:: groovy


    import groovy.transform.EqualsAndHashCode

    @EqualsAndHashCode
    class Person {
        String firstName
        String lastName
    }

    def p1 = new Person(firstName: 'Jack', lastName: 'Nicholson')
    def p2 = new Person(firstName: 'Jack', lastName: 'Nicholson')

    assert p1==p2
    assert p1.hashCode() == p2.hashCode()



这里有一些选项用于改变 ``@EqualsAndHashCode`` 的行为：

 （TBD）



@groovy.transform.TupleConstructor
++++++++++++++++++++++++++++++++++

