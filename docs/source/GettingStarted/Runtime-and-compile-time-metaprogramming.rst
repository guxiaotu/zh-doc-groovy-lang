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
