与 Java 不同之处
=================

``Groovy`` 试图使 Java 开发人员的学习曲线更加平滑。在设计 ``Groovy`` 时，我们一直在努力遵循最小意外原则，特别对于有 Java 开发背景的开发人员。

这里我们将列举 ``Java`` 与 ``Groovy`` 之间所有主要不同之处。


Default imports
-------------------

这些 ``packages`` 和 ``classes`` 是默认引入，你可以无需明确的引入，即可使用：

.. code-block::  groovy

	java.io.*

	java.lang.*

	java.math.BigDecimal

	java.math.BigInteger

	java.net.*

	java.util.*

	groovy.lang.*

	groovy.util.*


Multi-methods
-----------------

在 Groovy 中，在运行时选择需要被执行的方法。这被称为运行时路由 或 ``multi-methods`` 。这意味着在运行时，根据参数类型来确定调用方法。
在 Java 中，方法选择是在编译时，依赖声明的类型。

下面代码可以在 ``Java`` 和 ``Groovy`` 中编译通过，但是执行结果并不相同：

.. code-block:: groovy

	int method(String arg) {
	    return 1;
	}
	int method(Object arg) {
	    return 2;
	}
	Object o = "Object";
	int result = method(o);

Java 中的结果：

.. code-block:: groovy

	assertEquals(2, result);

下面是 Groovy 中的结果:

.. code-block:: groovy

	assertEquals(1, result);

在 Java 中的使用静态类型，这里 ``o`` 声明为 ``Object`` , 然而 Groovy 在方法调用时，选择具体参数类型。这里在运行时为 String ，则调用了上面的第一个方法。

Array initializers
------------------

在  Groovy 中， ``{ ... }`` 是为闭包预留. 这就意味着你无法使用这样的语句创建 list ： 

.. code-block:: groovy

	int[] array = { 1, 2,  3}

你需要这样写：

.. code-block:: groovy

	int[] array = [1,2,3]

Package scope visibility
------------------------

在 Groovy 中数据域上省略修饰符，与在 Java 中不同访问范围不同：

.. code-block:: groovy

	class Person {
	    String name
	}


Instead, it is used to create a property, that is to say a private field, an associated getter and an associated setter.

创建 ``package-private`` 成员变量，需要使用注解 ``@PackageScope``:

.. code-block:: groovy

	class Person {
	    @PackageScope String name
	}

ARM blocks
----------

ARM (Automatic Resource Management)  源自 Java 7， ``Groovy`` 中并没有支持。替代其， ``Groovy`` 中提供多种依赖闭包的方法来达到相同的效果并更加灵活便利，
例如：

.. code-block:: java

	Path file = Paths.get("/path/to/file");
	Charset charset = Charset.forName("UTF-8");
	try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
	    String line;
	    while ((line = reader.readLine()) != null) {
	        System.out.println(line);
	    }

	} catch (IOException e) {
	    e.printStackTrace();
	}

可以这样写：

.. code-block:: groovy

	new File('/path/to/file').eachLine('UTF-8') {
	   println it
	}

更接近 Java 的写法：	

.. code-block:: groovy

	new File('/path/to/file').withReader('UTF-8') { reader ->
	   reader.eachLine {
	       println it
	   }
	}

Inner classes
-------------

实现匿名内部类和嵌套类继承了 Java 中的思想，但是其与 Java 语言中的规范还有比较大的差别。这里的实现看起来很像 ``groovy.lang.Closure`` ，有一些好处及不同。
访问 private fields 和 方法成为一个问题，但一方面本地变量不必为 final 。

Static inner classes
^^^^^^^^^^^^^^^^^^^^

例如：

.. code-block:: groovy

	class A {
	    static class B {}
	}

	new A.B()

The usage of static inner classes is the best supported one. If you absolutely need an inner class, you should make it a static one.

Anonymous Inner Classes
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: groovy

	import java.util.concurrent.CountDownLatch
	import java.util.concurrent.TimeUnit

	CountDownLatch called = new CountDownLatch(1)

	Timer timer = new Timer()
	timer.schedule(new TimerTask() {
	    void run() {
	        called.countDown()
	    }
	}, 0)

	assert called.await(10, TimeUnit.SECONDS)

Creating Instances of Non-Static Inner Classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Java you can do this:

.. code-block:: java

	public class Y {
	    public class X {}
	    public X foo() {
	        return new X();
	    }
	    public static X createX(Y y) {
	        return y.new X();
	    }
	}

``Groovy`` 并不支持 ``y.new X()`` 句法。你可以这样写 ``new X(y)`` 来替代，就像下面的代码：

.. code-block:: groovy

	public class Y {
	    public class X {}
	    public X foo() {
	        return new X()
	    }
	    public static X createX(Y y) {
	        return new X(y)
	    }
	}

注意，Groovy 支持调用只有一个参数的方法，而不输入参数，这样参数的赋值为 ``null``。
在调用构造方法时，也是这样。当你使用 ``new X()`` 替代 ``new X(this)`` 会有危险，因为这种常规方式，我们还没有找到更好的方式来避免这种问题的出现。	

Lambdas
-------

Java 8 支持 lambdas 和 方法引用：

.. code-block:: java

	Runnable run = () -> System.out.println("Run");
	list.forEach(System.out::println);

Java 8 lambdas can be more or less considered as anonymous inner classes. Groovy doesn’t support that syntax, but has closures instead:
Java 8 lambdas 可以或多或少被认为是匿名内部类。``Groovy`` 中不支持这种语法，但可以使用闭包代替：

.. code-block:: groovy

	Runnable run = { println 'run' }
	list.each { println it } // or list.each(this.&println)

GStrings
--------

双引号字符被解释为 ``GStrings``， 当字符串中包含美元符号在 Groovy 和 Java 编译器中编译时可以能会引起编译失败或细微的差别。

通常， Groovy 将会自动转换 GString 与 String，如果接口上明确定义参数类型，需要注意， Java 接口接受一个 ``Object`` 参数，并检查其实际类型。

String and Character literals
-----------------------------

Groovy 中使用单引号定义 String, 双引号可以定义 String 或 GString, 取决于其是否插入字符。

.. code-block:: groovy

	assert 'c'.getClass()==String
	assert "c".getClass()==String
	assert "c${1}".getClass() in GString

Groovy 中声明变量为 ``char`` 将会自动转化单字符串为 ``char``。
当调用的方法参数为 ``char`` ，你需要之前确定类型的一致性。

.. code-block:: groovy

	char a='a'
	assert Character.digit(a, 16)==10 : 'But Groovy does boxing'
	assert Character.digit((char) 'a', 16)==10

	try {
	  assert Character.digit('a', 16)==10
	  assert false: 'Need explicit cast'
	} catch(MissingMethodException e) {
	}

Groovy 中支持两种方式转化 ``char``，这里有一些细微的不同当转化多字符字符串时。	
Groovy 转化方式容错更强，其将只转化第一个字符，当 ``C-style`` 转化将会抛出异常：

.. code-block:: groovy

	// for single char strings, both are the same
	assert ((char) "c").class==Character
	assert ("c" as char).class==Character

	// for multi char strings they are not
	try {
	  ((char) 'cx') == 'c'
	  assert false: 'will fail - not castable'
	} catch(GroovyCastException e) {
	}
	assert ('cx' as char) == 'c'
	assert 'cx'.asType(char) == 'c'

Primitives and wrappers
-----------------------

Groovy 中一切皆对象，原始类型会被自动包装为引用。
因为如此，它不会像 Java 那样类型拓阔优先于装箱，例如：
Because Groovy uses Objects for everything, it autowraps references to primitives. Because of this, it does not follow Java’s behavior of widening taking priority over boxing. Here’s an example using int

.. code-block:: groovy

	int i
	m(i)

	void m(long l) {                // <1>
	  println "in m(long)"
	}

	void m(Integer i) {            // <2>
	  println "in m(Integer)"
	}

<1> Java 中的调用路径，类型范围拓宽优先于拆箱
（This is the method that Java would call, since widening has precedence over unboxing.）

<2> Groovy 中的调用路径，其中所有原始类型引用会使用其包装类。
（This is the method Groovy actually calls, since all primitive references use their wrapper class.）


Behaviour of ==
---------------

Java 中 ``==`` 表明原始类型相等或对象地址相同。
Groovy 中 ``==`` 被翻译为 ``a.compareTo(b)==0``, 并且他们都是 ``Comparable`` 并且 ``a.equals(b)``;
为检查其地址相同，这里使用 ``is``  例如： ``a.is(b)``

Conversions
-----------

Java 自动拓宽或收窄：

Table 1. Java Conversions

+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| Conversions to     | boolean   | byte  | short  | char    | int    | long    | float   | double |
+--------------------+           |       |        |         |	     |         |         |        |		
| Conversions from   |           |       |        |         |        |         |         |        | 
+====================+===========+=======+========+=========+========+=========+=========+========+
| boolean            |    -      |  N    |  N     | N       |  N     |  N      |   N     |  N     |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| byte               | N         | -     |  Y     | C       | Y      | Y       | Y       | Y      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| short              | N         | C     |  -     | C       | Y      | Y       | Y       | Y      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| char               | N         | C     | C      | -       | Y      | Y       | Y       | Y      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| int                | N         | C     | C      | C       | -      | Y       | T       | Y      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| long               | N         | C     | C      | C       | C      | -       | T       | T      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| float              | N         | C     | C      | C       | C      | C       | -       | Y      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+
| double             | N         | C     | C      | C       | C      | C       | C       | -      |
+--------------------+-----------+-------+--------+---------+--------+---------+---------+--------+



.. note:: 

	* `Y` 表示 Java 自动转换， `C` 表示需要指定明确的类型转换，`T` 表示转换过程中有数据丢失，`N` 表示 Java 无法转换

Groovy expands greatly on this.

Table 2. Groovy Conversions

+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Conversions to     | boolean | Boolean | byte | Byte | short | Short | char | Character | int | Integer | long | Long | BigInteger | float | Float | double | Double | BigDecimal |
+--------------------+         |         |      |      |       |       |      |           |     |         |      |      |            |       |       |        |        |            |		
| Conversions from   |         |         |      |      |       |       |      |           |     |         |      |      |            |       |       |        |        |            | 
+====================+=========+=========+======+======+=======+=======+======+===========+=====+=========+======+======+============+=======+=======+========+========+============+
| boolean            | -       | B       | N    | N    | N     | N     | N    | N         | N   | N       | N    | N    | N          | N     | N     | N      | N      | N          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Boolean            | B       | -       | N    | N    | N     | N     | N    | N         | N   | N       | N    | N    | N          | N     | N     | N      | N      | N          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| byte               | T       | T       | -    | B    | Y     | Y     | Y    | D         | Y   | Y       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Byte               | T       | T       | B    | -    | Y     | Y     | Y    | D         | Y   | Y       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          |         
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| short              | T       | T       | D    | D    | -     | B     | Y    | D         | Y   | Y       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Short              | T       | T       | D    | T    | B     | -     | Y    | D         | Y   | Y       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| char               | T       | T       | Y    | D    | Y     | D     | -    | D         | Y   | D       | Y    | D    | D          | Y     | D     | Y      | D      | D          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| character          | T       | T       | D    | D    | D     | D     | D    | -         | D   | D       | D    | D    | D          | D     | D     | D      | D      | D          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| int                | T       | T       | D    | D    | D     | D     | Y    | D         | -   | B       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          | 
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Integer            | T       | T       | D    | D    | D     | D     | Y    | D         | B   | -       | Y    | Y    | Y          | Y     | Y     | Y      | Y      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| long               | T       | T       | D    | D    | D     | D     | Y    | D         | D   | D       | -    | B    | Y          | T     | T     | T      | T      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Long               | T       | T       | D    | D    | D     | T     | Y    | D         | D   | T       | B    | -    | Y          | T     | T     | T      | T      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| BigInteger         | T       | T       | D    | D    | D     | D     | D    | D         | D   | D       | D    | D    | -          | D     | D     | D      | D      | T          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| float              | T       | T       | D    | D    | D     | D     | T    | D         | D   | D       | D    | D    | D          | -     | B     | Y      | Y      | Y          |
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Float              | T       | T       | D    | T    | D     | T     | T    | D         | D   | T       | D    | T    | D          | B     | -     | Y      | Y      | Y          |	  
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| double             | T       | T       | D    | D    | D     | D     | T    | D         | D   | D       | D    | D    | D          | D     | D     | -      | B      | Y          | 
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| Double             | T       | T       | D    | T    | D     | T     | T    | D         | D   | T       | D    | T    | D          | D     | T     | B      | -      | Y          | 
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+
| BigDecimal         | T       | T       | D    | D    | D     | D     | D    | D         | D   | D       | D    | D    | D          | T     | D     | T      | D      | -          | 
+--------------------+---------+---------+------+------+-------+-------+------+-----------+-----+---------+------+------+------------+-------+-------+--------+--------+------------+


.. note:: 

	* `Y` 表示 Groovy 自动转换，`D` 表示 Groovy 可以在编译期间或明确类型时进行转换， `T` 表示转换过程中有数据丢失，`B` 表示装箱及拆箱操作， `N` 表示 Groovy 无法转换


The truncation uses Groovy Truth when converting to boolean/Boolean. Converting from a number to a character casts the Number.intvalue() to char. Groovy constructs BigInteger and BigDecimal using Number.doubleValue() when converting from a Float or Double, otherwise it constructs using toString(). Other conversions have their behavior defined by java.lang.Number.

新增关键字
-----------

Grooy 中新增了一些关键字，可以使用它们来命名变量

- as

- def

- in

- trait

default must be last in switch case
-----------------------------------

``default`` 必须在 ``switch/case`` 的结尾。在 Java 中 ``default`` 可以在 ``switch/case`` 中任何位置出现。
default must go at the end of the switch/case. While in Java the default can be placed anywhere in the switch/case, the default in Groovy is used more as an else than assigning a default case.

