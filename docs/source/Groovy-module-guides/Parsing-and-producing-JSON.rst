JSON 处理方式
================

	Groovy 中集成 JSON 与 对象之间转换的功能。
	在 ``groovy.json`` 包中的类专门用于 JSON 序列化及解析工作.

JsonSlurper
-----------

``JsonSlurper`` 用于将 JSON 字符串转换为 Groovy 数据结构，如：maps, lists, 以及原始类型 ``Integer`` , ``Double``, ``Boolean`` 和
``String``.

其类中有大量的 ``parse`` 以及如： ``parseText`` ，``parseFile`` 等方法。
下面的例子中将使用 ``parseText`` 方法来解析 JSON 字符串，并将其转换为 list 或 map 对象。
The class comes with a bunch of overloaded parse methods plus some special methods such as parseText, parseFile and others. For the next example we will use the parseText method. It parses a JSON String and recursively converts it to a list or map of objects. The other parse* methods are similar in that they return a JSON String but for different parameter types.

.. code-block:: groovy

	def jsonSlurper = new JsonSlurper()
	def object = jsonSlurper.parseText('{ "name": "John Doe" } /* some comment */')

	assert object instanceof Map
	assert object.name == 'John Doe'

这里的返回结果为 map , 可以像普通 Groovy 对象一样使用。 ``JsonSlurper`` 解析的 Json 字符串遵循 `ECMA-404 JSON 交换标准 <http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf>`_ ， 增加了对于注解及日期的支持。	

In addition to maps JsonSlurper supports JSON arrays which are converted to lists.
除了 map，``JsonSlurper`` 也支持将 JSON 数组转换为 lists .

.. code-block:: groovy

	def jsonSlurper = new JsonSlurper()
	def object = jsonSlurper.parseText('{ "myList": [4, 8, 15, 16, 23, 42] }')

	assert object instanceof Map
	assert object.myList instanceof List
	assert object.myList == [4, 8, 15, 16, 23, 42]

JSON 标准支持以下原始数据类型：string, number, object, true, false, null.
``JsonSlurper`` 会将它们转换为对应的 Groovy 类型。	

.. code-block:: groovy

	def jsonSlurper = new JsonSlurper()
	def object = jsonSlurper.parseText '''
	    { "simple": 123,
	      "fraction": 123.66,
	      "exponential": 123e12
	    }'''

	assert object instanceof Map
	assert object.simple.class == Integer
	assert object.fraction.class == BigDecimal
	assert object.exponential.class == BigDecimal

JsonSlurper 返回 Groovy 对象， 其返回结果符合 GPath 表达式。GPath 是一种功能很强的表达式语言，并通过 ``multiple slurpers`` 可以支持不同的
数据格式，例如：``XmlSlurper`` 用于支持 xml

想了解更多的细节，可以查看 `GPath表达式 <http://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#gpath_expressions>`_ 这一章节。

下面是 JSON 类型与对应的 Groovy 数据类型，对照表：

+-------------------+-------------------------------------------------------------+
| JSON              | Groovy                                                      |
+===================+=============================================================+
| Stirng            | java.lang.String                                            |
+-------------------+-------------------------------------------------------------+
| number            | java.lang.BigDecimal or java.lang.Integer                   |
+-------------------+-------------------------------------------------------------+
| object            | java.util.LinkedHashMap                                     |
+-------------------+-------------------------------------------------------------+
| array             | java.util.ArrayList                                         |
+-------------------+-------------------------------------------------------------+
| true              | true                                                        |
+-------------------+-------------------------------------------------------------+
| false             | false                                                       |
+-------------------+-------------------------------------------------------------+
| null              | null                                                        |
+-------------------+-------------------------------------------------------------+
| date              | java.util.Date ``yyyy-MM-dd’T’HH:mm:ssZ``                   |
+-------------------+-------------------------------------------------------------+


Whenever a value in JSON is null, JsonSlurper supplements it with the Groovy null value. This is in contrast to other JSON parsers that represent a null value with a library-provided singleton object.

Parser Variants
^^^^^^^^^^^^^^^

``JsonSlurper`` 附带了一些解析器实现。每种解析器适用于不同的需求，在特定的场景中，默认的解析器并不一定是最好的解决方案，
这里有一些解析器的简单介绍：

- ``JsonParserCharArray`` 解析器，将 JSON 字符串，基于字符数组方式处理，在处理过程中拷贝字符的子数组并处理（这种机制称为：``chopping``）.

The JsonFastParser is a special variant of the JsonParserCharArray and is the fastest parser. However, it is not the default parser for a reason. JsonFastParser is a so-called index-overlay parser. During parsing of the given JSON String it tries as hard as possible to avoid creating new char arrays or String instances. It keeps pointers to the underlying original character array only. In addition, it defers object creation as late as possible. If parsed maps are put into long-term caches care must be taken as the map objects might not be created and still consist of pointer to the original char buffer only. However, JsonFastParser comes with a special chop mode which dices up the char buffer early to keep a small copy of the original buffer. Recommendation is to use the JsonFastParser for JSON buffers under 2MB and keeping the long-term cache restriction in mind.
（``JsonFastParser`` 是基于 ``JsonParserCharArray`` 的扩展，它是最快速的解析器。其不是默认解析器，也是有一定原因。``JsonFastParser`` 是所谓指数覆盖解析器。在解析 JSON 字符串的时候，其尽可能避免创建新的字符数组或字符串。仅保持其指针在愿字符数组上。此外，也尽可能推迟对象的创建，）

- The JsonParserLax is a special variant of the JsonParserCharArray parser. It has similar performance characteristics as JsonFastParser but differs in that it isn’t exclusively relying on the ECMA-404 JSON grammar. For example it allows for comments, no quote strings etc.

- The JsonParserUsingCharacterSource is a special parser for very large files. It uses a technique called "character windowing" to parse large JSON files (large means files over 2MB size in this case) with constant performance characteristics.

``JsonSlurper`` 的默认解析器实现是 ``JsonParserCharArray``.
下表中列举了 JsonParserType 与 其实现的对照关系：

+-------------------------------------+------------------------------------------+
| Implementation                      | Constant                                 |
+=====================================+==========================================+
| JsonParserCharArray                 | JsonParserType#CHAR_BUFFER               |
+-------------------------------------+------------------------------------------+
| JsonFastParser                      | JsonParserType#INDEX_OVERLAY             |
+-------------------------------------+------------------------------------------+
| JsonParserLax                       | JJsonParserType#LAX                      |
+-------------------------------------+------------------------------------------+
| JsonParserUsingCharacterSource      | JsonParserType#CHARACTER_SOURCE          |
+-------------------------------------+------------------------------------------+


通过调用 ``JsonSlurper#setType()`` 来设置 ``JsonParserType`` 可以很方便的修改解析器的实现。

.. code-block:: groovy

	def jsonSlurper = new JsonSlurper(type: JsonParserType.INDEX_OVERLAY)
	def object = jsonSlurper.parseText('{ "myList": [4, 8, 15, 16, 23, 42] }')

	assert object instanceof Map
	assert object.myList instanceof List
	assert object.myList == [4, 8, 15, 16, 23, 42]

JsonOutput
----------

``JsonOutput`` 主要用于将  Groovy 对象序列化为 JSON 字符串。

JsonOutput is responsible for serialising Groovy objects into JSON strings. It can be seen as companion object to JsonSlurper, being a JSON parser.

``JsonOutput`` 中有一些重载，静态 ``toJson`` 方法。每种 ``toJson`` 实现都带有不同的参数。静态可以直接使用，或通过静态引入后使用。

``toJson`` 返回结果返回 JSON 格式的字符串。

.. code-block:: groovy

	def json = JsonOutput.toJson([name: 'John Doe', age: 42])

	assert json == '{"name":"John Doe","age":42}'

``JsonOutput`` 不仅仅支持原始类型，maps，lists，还可以支持 ``POGOs`` 序列化。

.. code-block:: groovy

	class Person { String name }

	def json = JsonOutput.toJson([ new Person(name: 'John'), new Person(name: 'Max') ])

	assert json == '[{"name":"John"},{"name":"Max"}]'

上面例子中，JSON 的默认输出没有使用 ``pretty printed``.

.. code-block:: groovy

	def json = JsonOutput.toJson([name: 'John Doe', age: 42])

	assert json == '{"name":"John Doe","age":42}'

	assert JsonOutput.prettyPrint(json) == '''\
	{
	    "name": "John Doe",
	    "age": 42
	}'''.stripIndent()

	
``prettyPrint`` 方法只有一个字符串输入参数，它不与 ``JsonOutput`` 中的其它方法结合使用，其可以处理任何的 JSON 格式字符串。

Groovy 中还可以使用 JsonBuilder 或 StreamingJsonBuilder 创建 JSON.
这两种方式提供 DSL 通过其定制对象的图谱，并将其转化为 JSON 。

For more details on builders, have a look at the builders chapter which covers both JsonBuilder and StreamingJsonBuilder.
如希望了解更多的细节，可以查看它们对应的章节，`JsonBuilder <http://docs.groovy-lang.org/latest/html/documentation/core-domain-specific-languages.html#_jsonbuilder>`_ 和 `StreamingJsonBuilder <http://docs.groovy-lang.org/latest/html/documentation/core-domain-specific-languages.html#_streamingjsonbuilder>`_