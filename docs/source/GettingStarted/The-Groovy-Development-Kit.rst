Groovy 开发包
=============

Working with IO
---------------------

Groovy 提供了大量的辅助方法用于 IO 处理。你可以使用标准的 JAVA 代码在 Groovy 中处理这些问题，Groovy 也提供了很多便利的方法处理文件，流，reader 等等

特别可以可以看一下，下面类中添加的方法：

- the ``java.io.File`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html
- the ``java.io.InputStream`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html
- the ``java.io.OutputStream`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html
- the ``java.io.Reader`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html
- the ``java.io.Writer`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html
- the ``java.nio.file.Path`` class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html

下面章节将主要介绍这些辅助方法使用的范例，但并不全面，如果需要了解更全面的内容，可以查看 `GDK API <http://groovy-lang.org/gdk.html>`_ .

读取文件
~~~~~~~~

第一个实例，看看如何打印出文件中的每行内容：

.. code-block:: groovy

	new File(baseDir, 'haiku.txt').eachLine { line ->
	    println line
	}


方法 eachLine 是由 Groovy 自动加入 File 类中。
如果你希望知道行号，可以使用下面方法：

.. code-block:: groovy

	new File(baseDir, 'haiku.txt').eachLine { line, nb ->
	    println "Line $nb: $line"
	}

如果有情况在 eachLine 方法中抛出异常，这方法将会确保资源是被合理关闭。Groovy 针对所有的 IO 处理都会遵循此中方式。

在一定条件下，我们使用 Reader ，同样也会感受到 Groovy 对资源自动化管理的好处。
下面的实例，将演示在异常发生时，reader 也将会被关闭。

.. code-block:: groovy

	def count = 0, MAXSIZE = 3
	new File(baseDir,"haiku.txt").withReader { reader ->
	    while (reader.readLine()) {
	        if (++count > MAXSIZE) {
	            throw new RuntimeException('Haiku should only have 3 verses')
	        }
	    }
	}

如果你希望将每一行内容存入 list， 你可以这么办：

.. code-block:: groovy

	def list = new File(baseDir, 'haiku.txt').collect {it}

或者你可以通过 `as` 操作符将内容转化为数组：

.. code-block:: groovy

	def array = new File(baseDir, 'haiku.txt') as String[]

将文件内容写入 byte[] 将要写多少代码？

Groovy 让这一切变的如此简单：

.. code-block:: groovy

	byte[] contents = file.bytes

使用 IO 不仅仅在处理文件。事实上，大量的操作依靠输入输出流，这也是 Groovy 增加了大量方法支持其，可以查看 `文档 <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html>`_

在下面的例子中，你可以很容易从文件中的获取 InputStream :

.. code-block:: groovy
	
	def is = new File(baseDir,'haiku.txt').newInputStream()
	// do something ...
	is.close()

然而你会看到，这样处理你将不得不去执行关闭 inputstream 。在 Groovy 中有更好的办法，使用 withInputStream 来帮助你处理这一切：

.. code-block:: groovy

	new File(baseDir,'haiku.txt').withInputStream { stream ->
	    // do something ...
	}


写文件
~~~~~~~~~~

当然在一些情况下，我们不仅仅要读文件，也需要来写文件。
我们有一个选择就是使用 Writer：

.. code-block:: groovy

	new File(baseDir,'haiku.txt').withWriter('utf-8') { writer ->
	    writer.writeLine 'Into the ancient pond'
	    writer.writeLine 'A frog jumps'
	    writer.writeLine 'Water’s sound!'
	}

但或者还有更简单的方法，使用 `<<` 操作符就可以满足需求：

.. code-block:: groovy
	
	new File(baseDir,'haiku.txt') << '''Into the ancient pond
	A frog jumps
	Water’s sound!'''

通常我们并不处理文本内容，所以你可以使用 Writer 或直接将字节写入，如：

.. code-block:: groovy

	file.bytes = [66,22,11]

你也可以直接操作输出流，向下面这样：

.. code-block:: groovy

	def os = new File(baseDir,'data.bin').newOutputStream()
	// do something ...
	os.close()

当然你同样看到这里的操作，还是需要关闭输出流。同样我们可以使用 withOutputStream 来更优雅的处理关闭与异常：

.. code-block:: groovy

	new File(baseDir,'data.bin').withOutputStream { stream ->
	    // do something ...
	}

遍历文件树
~~~~~~~~~~~~~~~~~~~~

在脚本中，遍历文件树，找到指定的文件并处理是比较常见的任务。Groovy 提供了多种方法进行处理。

列出文件目录下的所有文件：

.. code-block:: groovy

	dir.eachFile { file ->    //<1>
	    println file.name
	}
	dir.eachFileMatch(~/.*\.txt/) { file ->   //<2>
	    println file.name
	}

<1> 目录中找到的文件，执行闭包中代码

<2> 目录中满足条件文件，执行闭包中代码

通常情况下，你需要处理更深层次的文件，这种情况下可以使用 eachFileRecurse:

.. code-block:: groovy

	dir.eachFileRecurse { file ->    //<1>
	    println file.name
	}

	dir.eachFileRecurse(FileType.FILES) { file ->  //<2>
	    println file.name
	}

<1> 目录中递归查找到文件或目录后执行闭包中代码

<2> 目录中递归查找到文件后执行闭包中代码

对于更复杂的遍历技术你可以使用 `traverse` 方法，在这里需要你按指定遍历需求，进行相应的配置：

.. code-block:: groovy


	dir.traverse { file ->
	    if (file.directory && file.name=='bin') {
	        FileVisitResult.TERMINATE		//<1>
	    } else {
	        println file.name
	        FileVisitResult.CONTINUE  		//<2>
	    }

	}

<1> 如果当前文件为目录并且被命名为 bin ， 将停止便利

<2>	否则打印文件名，继续遍历

Data & objects
~~~~~~~~~~~~~~~~~~~~~~~~

在 JAVA 中通常使用 java.io.DataOutputStream & java.io.DataInputStream 来做对象的序列化与反序列化。
在 Groovy 中的使用将更加简单。
看看下面的例子，你可以使用下面代码将数据序列化至文件中，并在将其反序列化：

.. code-block:: groovy

	boolean b = true
	String message = 'Hello from Groovy'
	// Serialize data into a file
	file.withDataOutputStream { out ->
	    out.writeBoolean(b)
	    out.writeUTF(message)
	}
	// ...
	// Then read it back
	file.withDataInputStream { input ->
	    assert input.readBoolean() == b
	    assert input.readUTF() == message
	}

如果你数据对象实现 Serializable 接口，你可以使用下面关于对象的输出流来处理：

.. code-block:: groovy

	Person p = new Person(name:'Bob', age:76)
	// Serialize data into a file
	file.withObjectOutputStream { out ->
	    out.writeObject(p)
	}
	// ...
	// Then read it back
	file.withObjectInputStream { input ->
	    def p2 = input.readObject()
	    assert p2.name == p.name
	    assert p2.age == p.age
	}


执行外部进程
~~~~~~~~~~~~~~~~~~
前面章节描述了 Groovy 中文件，流，reader的处理方式。然而对于系统管理员或开发人员，他们通常都需要与系统相关进程进行交互。
Groovy 也提供了一种简单的方式来执行命令行进程。可以简单的使用一行字符串，然后调用 `exxcute()` 方法来执行。
例如：

.. code-block:: groovy

	def process = "ls -l".execute()     	//<1>
	println "Found text ${process.text}"	//<2>

<1> 执行 ls 命令

<2> 打印命令输出结果

``execute()`` 返回 ``java.lang.Process`` 实例，其可以通过 ``in/out/err streams`` 处理，并且检查进程退出值。
与上面命令一样，这里我们会对进程返回结果数据以流方式读取，每次读取一行数据：

.. code-block:: groovy

	def process = "ls -l".execute()		//<1>
	process.in.eachLine { line ->		//<2>
	    println line 					//<3>
	}

<1>	执行命令

<2>	按行读取进程输出数据

<3>	打印数据结果

值得注意的是与输入流对应的输出流命令，你可以发送数据至进程的 `out` .

请注意很多命令都是内建于 shell 中，如果你想在 windows 系统下列出当前目录中的文件，并且怎么写：

.. code-block:: groovy

	def process = "dir".execute()
	println "${process.text}"


你就会收到 IOException 描述：Cannot run program "dir": CreateProcess error=2, The system cannot find the file specified （无法执行 dir ： CreateProcess error=2, 系统无法找到制定的文件）

这是因为 ``dir`` 内建于 ``cmd.exe`` 中，并无法单独执行。
你需要这样写：

.. code-block:: groovy

	def process = "cmd /c dir".execute()
	println "${process.text}"

此外， ``java.lang.Process`` 中也隐藏了一些功能缺陷，值得我们关注。
在 javadoc 中有如下描述：

.. note:: 
	由于一些平台给输入输出流提供的缓存十分有限，从子进程中无法及时写入或读取，将导致子进程阻塞或死锁。
	Because some native platforms only provide limited buffer size for standard input and output streams, failure to promptly write the input stream or read the output stream of the subprocess may cause the subprocess to block, and even deadlock


正因为如此，Groovy提供一些额外的辅助方法，这使得流处理的过程更容易。
下面代码将演示如果如何吞掉你程序的输出（包括错误输出流）：

.. code-block:: groovy

	def p = "rm -f foo.tmp".execute([], tmpDir)
	p.consumeProcessOutput()
	p.waitFor()

这里有一些 ``consumeProcessOutput`` 的变化，在使用 ``StringBuffer``, ``InputStream``, ``OutputStream`` 等时。
具体的例子，请查看 `GDK API for java.lang.Process <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Process.html>`_ .

另外，这里还有 ``pipeTo`` 命令，可以将进程中输出流导入到其他进程的输入流中。

参考下面例子：

Pipes in action
+++++++++++++++++

.. code-block:: groovy

	proc1 = 'ls'.execute()
	proc2 = 'tr -d o'.execute()
	proc3 = 'tr -d e'.execute()
	proc4 = 'tr -d i'.execute()
	proc1 | proc2 | proc3 | proc4
	proc4.waitFor()
	if (proc4.exitValue()) {
	    println proc4.err.text
	} else {
	    println proc4.text
	}
	

Consuming errors
+++++++++++++++++

.. code-block:: groovy

	def sout = new StringBuilder()
	def serr = new StringBuilder()
	proc2 = 'tr -d o'.execute()
	proc3 = 'tr -d e'.execute()
	proc4 = 'tr -d i'.execute()
	proc4.consumeProcessOutput(sout, serr)
	proc2 | proc3 | proc4
	[proc2, proc3].each { it.consumeProcessErrorStream(serr) }
	proc2.withWriter { writer ->
	    writer << 'testfile.groovy'
	}
	proc4.waitForOrKill(1000)
	println "Standard output: $sout"
	println "Standard error: $serr"
	

Working with collections
-------------------------

Groovy 提供各种集合类型，包括 `lists <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Lists>`_， `maps <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Maps>`_， `ranges <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Ranges>`_。
其中大部份都是以 Java 集合类型为基础，并添加了一些附加方法在 `Groovy 开发包 <http://www.groovy-lang.org/gdk.html>`_ 中。

Lists
~~~~~~~~~~

List literals
^^^^^^^^^^^^^^

你可以像下面那样创建 lists 。``[]`` 用于创建空的列表。

.. code-block:: groovy

	def list = [5, 6, 7, 8]
	assert list.get(2) == 7
	assert list[2] == 7
	assert list instanceof java.util.List

	def emptyList = []
	assert emptyList.size() == 0
	emptyList.add(5)
	assert emptyList.size() == 1
	

每个 list 表达式都实现了 ``java.util.List``。
当让 lists 也可以用于创建新的 lists。

.. code-block:: groovy

	def list1 = ['a', 'b', 'c']
	//construct a new list, seeded with the same items as in list1
	def list2 = new ArrayList<String>(list1)

	assert list2 == list1 // == checks that each corresponding element is the same

	// clone() can also be called
	def list3 = list1.clone()
	assert list3 == list1
	

list 是有序序列对象：

.. code-block:: groovy

	def list = [5, 6, 7, 8]
	assert list.size() == 4
	assert list.getClass() == ArrayList     // the specific kind of list being used

	assert list[2] == 7                     // indexing starts at 0
	assert list.getAt(2) == 7               // equivalent method to subscript operator []
	assert list.get(2) == 7                 // alternative method

	list[2] = 9
	assert list == [5, 6, 9, 8,]           // trailing comma OK

	list.putAt(2, 10)                       // equivalent method to [] when value being changed
	assert list == [5, 6, 10, 8]
	assert list.set(2, 11) == 10            // alternative method that returns old value
	assert list == [5, 6, 11, 8]

	assert ['a', 1, 'a', 'a', 2.5, 2.5f, 2.5d, 'hello', 7g, null, 9 as byte]
	//objects can be of different types; duplicates allowed

	assert [1, 2, 3, 4, 5][-1] == 5             // use negative indices to count from the end
	assert [1, 2, 3, 4, 5][-2] == 4
	assert [1, 2, 3, 4, 5].getAt(-2) == 4       // getAt() available with negative index...
	try {
	    [1, 2, 3, 4, 5].get(-2)                 // but negative index not allowed with get()
	    assert false
	} catch (e) {
	    assert e instanceof ArrayIndexOutOfBoundsException
	}



列表用于布尔表式
^^^^^^^^^^^^^^^

.列表返回布尔值
+++++++++++++++++

.. code-block:: groovy

	assert ![]             // an empty list evaluates as false

	//all other lists, irrespective of contents, evaluate as true
	assert [1] && ['a'] && [0] && [0.0] && [false] && [null]


列表迭代器
^^^^^^^^^^^^

可使用 each 和 eachWithIndex 方法，用于列表上元素的迭代操作，可参考下面代码：
.. code-block:: groovy

	[1, 2, 3].each {
	    println "Item: $it" // `it` is an implicit parameter corresponding to the current element
	}
	['a', 'b', 'c'].eachWithIndex { it, i -> // `it` is the current element, while `i` is the index
	    println "$i: $it"
	}

在迭代的基础上，通常需要将列表中的元素经过一定转换，构建新的列表。这种操作称为 mapping ，在 Groovy 中可以使用 `collect` 方法:

.. code-block:: groovy

	assert [1, 2, 3].collect { it * 2 } == [2, 4, 6]

	// shortcut syntax instead of collect
	assert [1, 2, 3]*.multiply(2) == [1, 2, 3].collect { it.multiply(2) }

	def list = [0]
	// it is possible to give `collect` the list which collects the elements
	assert [1, 2, 3].collect(list) { it * 2 } == [0, 2, 4, 6]
	assert list == [0, 2, 4, 6]



操作列表
^^^^^^^^^^^^^

过滤和查询
+++++++++++++++++

`Groovy 开发包 <http://www.groovy-lang.org/gdk.html>`_ 中，集合上包含了很多方法用于增强标准集合的处理，可以看下面的例子:

.. code-block:: groovy

	assert [1, 2, 3].find { it > 1 } == 2           // find 1st element matching criteria
	assert [1, 2, 3].findAll { it > 1 } == [2, 3]   // find all elements matching critieria
	assert ['a', 'b', 'c', 'd', 'e'].findIndexOf {      // find index of 1st element matching criteria
	    it in ['c', 'e', 'g']
	} == 2

	assert ['a', 'b', 'c', 'd', 'c'].indexOf('c') == 2  // index returned
	assert ['a', 'b', 'c', 'd', 'c'].indexOf('z') == -1 // index -1 means value not in list
	assert ['a', 'b', 'c', 'd', 'c'].lastIndexOf('c') == 4

	assert [1, 2, 3].every { it < 5 }               // returns true if all elements match the predicate
	assert ![1, 2, 3].every { it < 3 }
	assert [1, 2, 3].any { it > 2 }                 // returns true if any element matches the predicate
	assert ![1, 2, 3].any { it > 3 }

	assert [1, 2, 3, 4, 5, 6].sum() == 21                // sum anything with a plus() method
	assert ['a', 'b', 'c', 'd', 'e'].sum {
	    it == 'a' ? 1 : it == 'b' ? 2 : it == 'c' ? 3 : it == 'd' ? 4 : it == 'e' ? 5 : 0
	    // custom value to use in sum
	} == 15
	assert ['a', 'b', 'c', 'd', 'e'].sum { ((char) it) - ((char) 'a') } == 10
	assert ['a', 'b', 'c', 'd', 'e'].sum() == 'abcde'
	assert [['a', 'b'], ['c', 'd']].sum() == ['a', 'b', 'c', 'd']

	// an initial value can be provided
	assert [].sum(1000) == 1000
	assert [1, 2, 3].sum(1000) == 1006

	assert [1, 2, 3].join('-') == '1-2-3'           // String joining
	assert [1, 2, 3].inject('counting: ') {
	    str, item -> str + item                     // reduce operation
	} == 'counting: 123'
	assert [1, 2, 3].inject(0) { count, item ->
	    count + item
	} == 6
	

下面是在 Groovy 代码中惯用的在集合中查找最大值与最小值的方法:

.. code-block:: groovy

	def list = [9, 4, 2, 10, 5]
	assert list.max() == 10
	assert list.min() == 2

	// we can also compare single characters, as anything comparable
	assert ['x', 'y', 'a', 'z'].min() == 'a'

	// we can use a closure to specify the sorting behaviour
	def list2 = ['abc', 'z', 'xyzuvw', 'Hello', '321']
	assert list2.max { it.size() } == 'xyzuvw'
	assert list2.min { it.size() } == 'z'


除了使用闭包，也可以使用 ``Comparator`` 定义比较方法：

.. code-block:: groovy

	Comparator mc = { a, b -> a == b ? 0 : (a < b ? -1 : 1) }

	def list = [7, 4, 9, -6, -1, 11, 2, 3, -9, 5, -13]
	assert list.max(mc) == 11
	assert list.min(mc) == -13

	Comparator mc2 = { a, b -> a == b ? 0 : (Math.abs(a) < Math.abs(b)) ? -1 : 1 }


	assert list.max(mc2) == -13
	assert list.min(mc2) == -1

	assert list.max { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -13
	assert list.min { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -1
	

.添加／删除列表元素
+++++++++++++++++

我们可以使用 ``[]`` 声明一个空列表并且使用 ``<<`` 向列表中追加元素：

.. code-block:: groovy

	def list = []
	assert list.empty

	list << 5
	assert list.size() == 1

	list << 7 << 'i' << 11
	assert list == [5, 7, 'i', 11]

	list << ['m', 'o']
	assert list == [5, 7, 'i', 11, ['m', 'o']]

	//first item in chain of << is target list
	assert ([1, 2] << 3 << [4, 5] << 6) == [1, 2, 3, [4, 5], 6]

	//using leftShift is equivalent to using <<
	assert ([1, 2, 3] << 4) == ([1, 2, 3].leftShift(4))


你可以通过多种方式向列表中添加元素:

.. code-block:: groovy

	assert [1, 2] + 3 + [4, 5] + 6 == [1, 2, 3, 4, 5, 6]
	// equivalent to calling the `plus` method
	assert [1, 2].plus(3).plus([4, 5]).plus(6) == [1, 2, 3, 4, 5, 6]

	def a = [1, 2, 3]
	a += 4      // creates a new list and assigns it to `a`
	a += [5, 6]
	assert a == [1, 2, 3, 4, 5, 6]

	assert [1, *[222, 333], 456] == [1, 222, 333, 456]
	assert [*[1, 2, 3]] == [1, 2, 3]
	assert [1, [2, 3, [4, 5], 6], 7, [8, 9]].flatten() == [1, 2, 3, 4, 5, 6, 7, 8, 9]

	def list = [1, 2]
	list.add(3)
	list.addAll([5, 4])
	assert list == [1, 2, 3, 5, 4]

	list = [1, 2]
	list.add(1, 3) // add 3 just before index 1
	assert list == [1, 3, 2]

	list.addAll(2, [5, 4]) //add [5,4] just before index 2
	assert list == [1, 3, 5, 4, 2]

	list = ['a', 'b', 'z', 'e', 'u', 'v', 'g']
	list[8] = 'x' // the [] operator is growing the list as needed
	// nulls inserted if required
	assert list == ['a', 'b', 'z', 'e', 'u', 'v', 'g', null, 'x']


相比较 ``<<`` , ``+`` 操作符将创建新的 list 对象，这样的将会带来性能问题，通常会弱化这样的操作：

`Groovy 开发包 <http://www.groovy-lang.org/gdk.html>`_ 也提供通过列表中的值来删除列表的方法：

.. code-block:: groovy

	assert ['a','b','c','b','b'] - 'c' == ['a','b','b','b']
	assert ['a','b','c','b','b'] - 'b' == ['a','c']
	assert ['a','b','c','b','b'] - ['b','c'] == ['a']

	def list = [1,2,3,4,3,2,1]
	list -= 3           // creates a new list by removing `3` from the original one
	assert list == [1,2,4,2,1]
	assert ( list -= [2,4] ) == [1,1]
	

也可以通过列表索引删除元素:

.. code-block:: groovy

	def list = [1,2,3,4,5,6,2,2,1]
	assert list.remove(2) == 3          // remove the third element, and return it
	assert list == [1,2,4,5,6,2,2,1]


如需要删除列表中与指定值相同的第一个元素，而不是所有，可以调用 `remove` 方法 ：

.. code-block:: groovy

	def list= ['a','b','c','b','b']
	assert list.remove('c')             // remove 'c', and return true because element removed
	assert list.remove('b')             // remove first 'b', and return true because element removed

	assert ! list.remove('z')           // return false because no elements removed
	assert list == ['a','b','b']


删除列表中的所有元素，可以使用 ``clear`` 方法：

.. code-block:: groovy

	def list= ['a',2,'c',4]
	list.clear()
	assert list == []


.Set operations
+++++++++++++++++

`Groovy 开发包 <http://www.groovy-lang.org/gdk.html>`_ 中有方法可以很方便的操作集合。

.. code-block:: groovy

	assert 'a' in ['a','b','c']             // returns true if an element belongs to the list
	assert ['a','b','c'].contains('a')      // equivalent to the `contains` method in Java
	assert [1,3,4].containsAll([1,4])       // `containsAll` will check that all elements are found

	assert [1,2,3,3,3,3,4,5].count(3) == 4  // count the number of elements which have some value
	assert [1,2,3,3,3,3,4,5].count {
	    it%2==0                             // count the number of elements which match the predicate
	} == 2

	assert [1,2,4,6,8,10,12].intersect([1,3,6,9,12]) == [1,6,12]  //  相交

	assert [1,2,3].disjoint( [4,6,9] )  // 不相交
	assert ![1,2,3].disjoint( [2,4,6] )


.排序
+++++++++++++++++

使用集合类型，通常都需要进行排序。Groovy 中提供了多种列表的排序方式，包括闭包比较方式，如下面例子:

.. code-block:: groovy

	assert [6, 3, 9, 2, 7, 1, 5].sort() == [1, 2, 3, 5, 6, 7, 9]

	def list = ['abc', 'z', 'xyzuvw', 'Hello', '321']
	assert list.sort {
	    it.size()
	} == ['z', 'abc', '321', 'Hello', 'xyzuvw']

	def list2 = [7, 4, -6, -1, 11, 2, 3, -9, 5, -13]
	assert list2.sort { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } ==
	        [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

	Comparator mc = { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 }

	// JDK 8+ only
	// list2.sort(mc)
	// assert list2 == [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

	def list3 = [6, -3, 9, 2, -7, 1, 5]

	Collections.sort(list3)
	assert list3 == [-7, -3, 1, 2, 5, 6, 9]

	Collections.sort(list3, mc)
	assert list3 == [1, 2, -3, 5, 6, -7, 9]
	

.复制元素
++++++++++++++

Groovy 开发包中也提供了操作符重载复制元素的方法：

.. code-block:: groovy

	assert [1, 2, 3] * 3 == [1, 2, 3, 1, 2, 3, 1, 2, 3]
	assert [1, 2, 3].multiply(2) == [1, 2, 3, 1, 2, 3]
	assert Collections.nCopies(3, 'b') == ['b', 'b', 'b']

	// nCopies from the JDK has different semantics than multiply for lists
	assert Collections.nCopies(2, [1, 2]) == [[1, 2], [1, 2]] //not [1,2,1,2]



Maps
-----------------

Map literals
~~~~~~~~~~~~~~~~~~~~~~~~
在 Groovy 中可以通过语法 ``[:]`` 创建 ``map``:

.. code-block:: groovy

	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.get('name') == 'Gromit'
	assert map.get('id') == 1234
	assert map['name'] == 'Gromit'
	assert map['id'] == 1234
	assert map instanceof java.util.Map

	def emptyMap = [:]
	assert emptyMap.size() == 0
	emptyMap.put("foo", 5)
	assert emptyMap.size() == 1
	assert emptyMap.get("foo") == 5

Map 的 key 默认为 String 类型: [a:1] 与 ['a':1] 等价。
Map keys are strings by default: [a:1] is equivalent to ['a':1]。
如果变量名为 a ，你想用 a 的值作为 map 的 key， 这样就会出现混淆。对于这种情况，就需要添加括号，向下面代码中描述：

.. code-block:: groovy

	def a = 'Bob'
	def ages = [a: 43]
	assert ages['Bob'] == null // `Bob` is not found
	assert ages['a'] == 43     // because `a` is a literal!

	ages = [(a): 43]            // now we escape `a` by using parenthesis
	assert ages['Bob'] == 43   // and the value is found!


Map 中可以通过 clone 方法获取一个新的拷贝:

.. code-block:: groovy

	def map = [
	        simple : 123,
	        complex: [a: 1, b: 2]
	]
	def map2 = map.clone()
	assert map2.get('simple') == map.get('simple')
	assert map2.get('complex') == map.get('complex')
	map2.get('complex').put('c', 3)
	assert map.get('complex').get('c') == 3

上面例子是关于 map 的浅拷贝。

Map property notation
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Maps 可以像 beans 那样通过属性访问符，获取或设置属性值，只要 key 为 map 中有效字符：

.. code-block:: groovy

	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.name == 'Gromit'     // can be used instead of map.get('Gromit')
	assert map.id == 1234

	def emptyMap = [:]
	assert emptyMap.size() == 0
	emptyMap.foo = 5
	assert emptyMap.size() == 1
	assert emptyMap.foo == 5

.. note::
`map.foo` 会在 map 中查找 key 为 foo 的值。这意味着 foo.class 将会返回 `null` ，因为在 map 中不能以 `class` 作为 key。如果需要得到这个 key 的 `class` ， 你需要使用 getClass() ：

.. code-block:: groovy

	def map = [name: 'Gromit', likes: 'cheese', id: 1234]
	assert map.class == null
	assert map.get('class') == null
	assert map.getClass() == LinkedHashMap // this is probably what you want

	map = [1      : 'a',
	       (true) : 'p',
	       (false): 'q',
	       (null) : 'x',
	       'null' : 'z']
	assert map.containsKey(1) // 1 is not an identifier so used as is
	assert map.true == null
	assert map.false == null
	assert map.get(true) == 'p'
	assert map.get(false) == 'q'
	assert map.null == 'z'
	assert map.get(null) == 'x'


maps 上的迭代
~~~~~~~~~~~~~~
在 Groovy 开发包中，惯用的迭代方法为 ``each`` 和 ``eachWithIndex``。
值得注意的是，map 创建是有序的，如果在 map 上使用的迭代，其返回的实体顺序与加入时顺序一致。

.. code-block:: groovy

	def map = [
	        Bob  : 42,
	        Alice: 54,
	        Max  : 33
	]

	// `entry` is a map entry
	map.each { entry ->
	    println "Name: $entry.key Age: $entry.value"
	}

	// `entry` is a map entry, `i` the index in the map
	map.eachWithIndex { entry, i ->
	    println "$i - Name: $entry.key Age: $entry.value"
	}

	// Alternatively you can use key and value directly
	map.each { key, value ->
	    println "Name: $key Age: $value"
	}

	// Key, value and i as the index in the map
	map.eachWithIndex { key, value, i ->
	    println "$i - Name: $key Age: $value"
	}


操作 Map
~~~~~~~~

.添加／删除元素
^^^^^^^^^^^^^
向 map 中添加元素，可以使用 ``put`` ， 下标操作符或 ``putAll`` 方法:

.. code-block:: groovy

	def defaults = [1: 'a', 2: 'b', 3: 'c', 4: 'd']
	def overrides = [2: 'z', 5: 'x', 13: 'x']

	def result = new LinkedHashMap(defaults)
	result.put(15, 't')
	result[17] = 'u'
	result.putAll(overrides)
	assert result == [1: 'a', 2: 'z', 3: 'c', 4: 'd', 5: 'x', 13: 'x', 15: 't', 17: 'u']


调用 ``clear`` 方法可以清除 ``map``  中的所有元素：

.. code-block:: groovy

	def m = [1:'a', 2:'b']
	assert m.get(1) == 'a'
	m.clear()
	assert m == [:]


Maps 生成使用 object 的 equals 和 hashcode 方法。这意味着你绝不可使用 hash code 会变化的对象作为 key， 否则你将无法获取对应的值。
这里需要提到是，你绝不可使用 GString 作为 map 的 key，因为 GString 的 hash code 与 String 类型的 hash code 不一致：

.. code-block:: groovy

	def key = 'some key'
	def map = [:]
	def gstringKey = "${key.toUpperCase()}"
	map.put(gstringKey,'value')
	assert map.get('SOME KEY') == null

.Keys, values and entries
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: groovy

	def map = [1:'a', 2:'b', 3:'c']

	def entries = map.entrySet()
	entries.each { entry ->
	  assert entry.key in [1,2,3]
	  assert entry.value in ['a','b','c']
	}

	def keys = map.keySet()
	assert keys == [1,2,3] as Set


Mutating values returned by the view (be it a map entry, a key or a value) is highly discouraged because success of the operation directly depends on the type of the map being manipulated. In particular, Groovy relies on collections from the JDK that in general make no guarantee that a collection can safely be manipulated through keySet, entrySet, or values.

.过滤／查询
^^^^^^^^^^^^
Groovy 开发包中包括过滤，查询，收集方法与 http://www.groovy-lang.org/groovy-dev-kit.html#List-Filtering[list] 中相似：
The Groovy development kit contains filtering, searching and collecting methods similar to those found for lists:

.. code-block:: groovy

	def people = [
	    1: [name:'Bob', age: 32, gender: 'M'],
	    2: [name:'Johnny', age: 36, gender: 'M'],
	    3: [name:'Claire', age: 21, gender: 'F'],
	    4: [name:'Amy', age: 54, gender:'F']
	]

	def bob = people.find { it.value.name == 'Bob' } // find a single entry
	def females = people.findAll { it.value.gender == 'F' }

	// both return entries, but you can use collect to retrieve the ages for example
	def ageOfBob = bob.value.age
	def agesOfFemales = females.collect {
	    it.value.age
	}

	assert ageOfBob == 32
	assert agesOfFemales == [21,54]

	// but you could also use a key/pair value as the parameters of the closures
	def agesOfMales = people.findAll { id, person ->
	    person.gender == 'M'
	}.collect { id, person ->
	    person.age
	}
	assert agesOfMales == [32, 36]

	// `every` returns true if all entries match the predicate
	assert people.every { id, person ->
	    person.age > 18
	}

	// `any` returns true if any entry matches the predicate

	assert people.any { id, person ->
	    person.age == 54
	}


.分组
^^^^^^

.. code-block:: groovy

	assert ['a', 7, 'b', [2, 3]].groupBy {
	    it.class
	} == [(String)   : ['a', 'b'],
	      (Integer)  : [7],
	      (ArrayList): [[2, 3]]
	]

	assert [
	        [name: 'Clark', city: 'London'], [name: 'Sharma', city: 'London'],
	        [name: 'Maradona', city: 'LA'], [name: 'Zhang', city: 'HK'],
	        [name: 'Ali', city: 'HK'], [name: 'Liu', city: 'HK'],
	].groupBy { it.city } == [
	        London: [[name: 'Clark', city: 'London'],
	                 [name: 'Sharma', city: 'London']],
	        LA    : [[name: 'Maradona', city: 'LA']],
	        HK    : [[name: 'Zhang', city: 'HK'],
	                 [name: 'Ali', city: 'HK'],
	                 [name: 'Liu', city: 'HK']],
	]


Ranges
~~~~~~~~~~~~~~~~~~

``Ranges`` 允许你创建连续值的列表对象。Ranges 可以像 List 一样使用，Range 继承 ``java.util.List``。
``Ranges`` 使用 ``..`` 定义闭区间。
``Ranges`` 使用 ``..<`` 定义半开区间，其包含第一个值，不包含最后一个值。

.. code-block:: groovy

	// an inclusive range
	def range = 5..8
	assert range.size() == 4
	assert range.get(2) == 7
	assert range[2] == 7
	assert range instanceof java.util.List
	assert range.contains(5)
	assert range.contains(8)

	// lets use a half-open range
	range = 5..<8
	assert range.size() == 3
	assert range.get(2) == 7
	assert range[2] == 7
	assert range instanceof java.util.List
	assert range.contains(5)
	assert !range.contains(8)

	//get the end points of the range without using indexes
	range = 1..10
	assert range.from == 1
	assert range.to == 10


int 型 ranges 实现十分有效，创建一个轻量的 java 对象可以包括启始值与终止值。

``Ranges`` 可以在任何实现 ``java.lang.Comparable`` 接口的 java 对象上使用，可以通过 next() 和 previous() 方法分别返回范围中，当前数据前后的值。例如，你可以创建一个 String 有序范围:

.. code-block:: groovy

	// an inclusive range
	def range = 'a'..'d'
	assert range.size() == 4
	assert range.get(2) == 'c'
	assert range[2] == 'c'
	assert range instanceof java.util.List
	assert range.contains('a')
	assert range.contains('d')
	assert !range.contains('e')


你可使用 ``for`` 循环：

.. code-block:: groovy

	for (i in 1..10) {
	    println "Hello ${i}"
	}


也可以使用 Groovy 中比较惯用的方式，使用 each 方法进行迭代操作：

.. code-block:: groovy

	(1..10).each { i ->
	    println "Hello ${i}"
	}


Ranges 可以在 ``switch`` 中使用：

.. code-block:: groovy

	switch (years) {
	    case 1..10: interestRate = 0.076; break;
	    case 11..25: interestRate = 0.052; break;
	    default: interestRate = 0.037;
	}


集合语法增强
~~~~~~~~~~~~~

GPath support
^^^^^^^^^^^^^^^
属性符号对 lists 和 maps 的支持， Groovy 提供了语法糖用于处理嵌套的集合对象，可参考下面例子：

.. code-block:: groovy

	def listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22]]
	assert listOfMaps.a == [11, 21] //GPath notation
	assert listOfMaps*.a == [11, 21] //spread dot notation

	listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22], null]
	assert listOfMaps*.a == [11, 21, null] // caters for null values
	assert listOfMaps*.a == listOfMaps.collect { it?.a } //equivalent notation
	// But this will only collect non-null values
	assert listOfMaps.a == [11,21]

Spread operator
^^^^^^^^^^^^^^^^^

``Spread operator`` 被用于集合内部的集合操作。这种语法糖可以避免使用 `putAll` ，有利于在单行内代码实现。

.. code-block:: groovy

	assert [ 'z': 900,
	         *: ['a': 100, 'b': 200], 'a': 300] == ['a': 300, 'b': 200, 'z': 900]
	//spread map notation in map definition
	assert [*: [3: 3, *: [5: 5]], 7: 7] == [3: 3, 5: 5, 7: 7]

	def f = { [1: 'u', 2: 'v', 3: 'w'] }
	assert [*: f(), 10: 'zz'] == [1: 'u', 10: 'zz', 2: 'v', 3: 'w']
	//spread map notation in function arguments
	f = { map -> map.c }
	assert f(*: ['a': 10, 'b': 20, 'c': 30], 'e': 50) == 30

	f = { m, i, j, k -> [m, i, j, k] }
	//using spread map notation with mixed unnamed and named arguments
	assert f('e': 100, *[4, 5], *: ['a': 10, 'b': 20, 'c': 30], 6) ==
	        [["e": 100, "b": 20, "c": 30, "a": 10], 4, 5, 6]


The star-dot `*.' operator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`*.` 操作符用于集合上所有元素调用方法或属性：

.. code-block:: groovy

	assert [1, 3, 5] == ['a', 'few', 'words']*.size()

	class Person {
	    String name
	    int age
	}
	def persons = [new Person(name:'Hugo', age:17), new Person(name:'Sandra',age:19)]
	assert [17, 19] == persons*.age


下标操作符用于切片
^^^^^^^^^^^^^^^
你可以使用下标表达式在 lists，maps，arrays 上进行索引定位。
字符串被看作一种特殊的集合：

.. code-block:: groovy

	def text = 'nice cheese gromit!'
	def x = text[2]

	assert x == 'c'
	assert x.class == String

	def sub = text[5..10]
	assert sub == 'cheese'

	def list = [10, 11, 12, 13]
	def answer = list[2,3]
	assert answer == [12,13]


注意你可以使用 ranges 提取集合的一部分：

.. code-block:: groovy

	list = 100..200
	sub = list[1, 3, 20..25, 33]
	assert sub == [101, 103, 120, 121, 122, 123, 124, 125, 133]

下标操作符可以用于更新集合（集合类型为可变集合）:

.. code-block:: groovy

	list = ['a','x','x','d']
	list[1..2] = ['b','c']
	assert list == ['a','b','c','d']


下标可以使用负数，可以方便从集合末尾开发提取：
你可以使用负数下标从末尾计算 List ，array ， String 等

.. code-block:: groovy

	text = "nice cheese gromit!"
	x = text[-1]
	assert x == "!"

	def name = text[-7..-2]
	assert name == "gromit"


你可以使用逆向范围，将集合结果进行反转。

.. code-block:: groovy

	text = "nice cheese gromit!"
	name = text[3..1]
	assert name == "eci"

增强集合方法
~~~~~~~~~~~~~~~

对于 `lists <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Lists>`_ , `maps <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Maps>`_ 和 `ranges <http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Ranges>`_ ， Groovy 提供了大量扩展方法用于过滤，收集，分组以及计算等等，其可以在自身上执行并且更容易使用迭代处理。

具体的内容可以阅读 `Groovy 开发包 API <http://www.groovy-lang.org/gdk.html>`_ 文档：

- methods added to Iterable can be found `here <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Iterable.html>`_
- methods added to Iterator can be found `here <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Iterator.html>`_
- methods added to Collection can be found `here <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Collection.html>`_
- methods added to List can be found `here <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/List.html>`_
- methods added to Map can be found `here <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Map.html>`_


Handy utilities (工具集)
-------------------------

ConfigSlurper
~~~~~~~~~~~~~

`ConfigSlurper` 是用于读取从 Groovy script 中读取配置信息。类似于 Java 中的 ``*.properties`` 文件， ``ConfigSlurper`` 中可以使用点符号。它也可以在闭包范围内定义内容以及任何的对象类型。

.. code-block:: groovy

	def config = new ConfigSlurper().parse('''
	    app.date = new Date()  			//<1>
	    app.age  = 42
	    app {							//<2>
	        name = "Test${42}"
	    }
	''')

	assert config.app.date instanceof Date
	assert config.app.age == 42
	assert config.app.name == 'Test42'


<1> 使用点符号

<2> 使用闭包替代点符号

从上面的例子可以看到，``parse`` 返回 ``groovy.util.ConfigObject`` 实例。 ``ConfigObject`` 实现了 ``java.util.Map`` 返回配置值或 ``ConfigObject`` 实例，但绝不会为 ``null``.

.. code-block:: groovy

	def config = new ConfigSlurper().parse('''
	    app.date = new Date()
	    app.age  = 42
	    app.name = "Test${42}"
	''')

	assert config.test != null   	//<1>


<1>  `config.test` 并没有定义，但是当其被调用时返回 `ConfigObject` 实例。

当配置中的变量名称中包含点时，可以使用单引号或双引号标注。

.. code-block:: groovy

	def config = new ConfigSlurper().parse('''
	    app."person.age"  = 42
	''')

	assert config.app."person.age" == 42


``ConfigSlurper`` 可以支持按环境配置。``environments`` 中包含各个环境组成，被用于闭包中。
让我们创建一个指定开发环境配置值。可以使用 `ConfigSlurper(String)` 构造方法指定目标环境。

.. code-block:: groovy

	def config = new ConfigSlurper('development').parse('''
	  environments {
	       development {
	           app.port = 8080
	       }

	       test {
	           app.port = 8082
	       }

	       production {
	           app.port = 80
	       }
	  }
	''')

	assert config.app.port == 8080


``ConfigSlurper`` 中的 ``environments`` 并没有强制指定特殊的环境名称。其可以由代码自行定义。
``environments`` 是内置方法，``registerConditionalBlock`` 方法可以用来注册 ``environments`` 以外的，其他的方法名称。

.. code-block:: groovy

	def slurper = new ConfigSlurper()
	slurper.registerConditionalBlock('myProject', 'developers')		// <1>

	def config = slurper.parse('''
	  sendMail = true

	  myProject {
	       developers {
	           sendMail = false
	       }
	  }
	''')

	assert !config.sendMail

<1> 向 `ConfigSlurper` 注册一个新的块

为与 Java 集成, ``toProperties`` 方法用于将 ``ConfigObject`` 转化为 ``java.util.Properties`` 对象，可以存储为 ``*.properties`` 文本文件。需要注意，配置的值将被转换为字符串类型，当将它们加入到 ``Properties`` 中时。

.. code-block:: groovy

	def config = new ConfigSlurper().parse('''
	    app.date = new Date()
	    app.age  = 42
	    app {
	        name = "Test${42}"
	    }
	''')

	def properties = config.toProperties()

	assert properties."app.date" instanceof String
	assert properties."app.age" == '42'
	assert properties."app.name" == 'Test42'


``Expando`` 类用于创建动态扩展对象。

每个 `Expando` 实例均为独立，动态创建的，可以在运行时动态扩展属性。

.. code-block:: groovy

	def expando = new Expando()
	expando.name = 'John'

	assert expando.name == 'John'


当其动态属性注册为闭包代码块，一旦调用其将按照方法调用执行。

.. code-block:: groovy

	def expando = new Expando()
	expando.toString = { -> 'John' }
	expando.say = { String s -> "John says: ${s}" }

	assert expando as String == 'John'
	assert expando.say('Hi') == 'John says: Hi'

监听 list, map and set
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Groovy 中自带 lists, map, sets 的观察器。这些集合对象，在添加，删除，或修改元素时，均为触发 ``java.beans.PropertyChangeEvent`` 事件。

需要注意的是，其不仅仅是发送信号，还会将留存属性值的历史变化。

根据不同的操作，这些集合将触发特定的 ``PropertyChangeEvent`` 类型。例如，添加一个元素到集合中，将触发 `ObservableList.ElementAddedEvent` 事件。

.. code-block:: groovy

	def event 					// <1>
	def listener = {
	    if (it instanceof ObservableList.ElementEvent)  {  		//<2>
	        event = it
	    }
	} as PropertyChangeListener


	def observable = [1, 2, 3] as ObservableList 				//<3>
	observable.addPropertyChangeListener(listener) 				//<4>

	observable.add 42 											//<5>

	assert event instanceof ObservableList.ElementAddedEvent

	def elementAddedEvent = event as ObservableList.ElementAddedEvent
	assert elementAddedEvent.changeType == ObservableList.ChangeType.ADDED
	assert elementAddedEvent.index == 3
	assert elementAddedEvent.oldValue == null
	assert elementAddedEvent.newValue == 42


<1> 声明一个 ``PropertyChangeEventListener`` 用于监听触发事件

<2> ``ObservableList.ElementEvent`` 对于监听器事件

<3> 注册监听器

<4> 创建一个可观察列表

<5> 触发添加事件

请注意，添加一个元素会触发两个事件。第一个是 ``ObservableList.ElementAddedEvent`` ，第二个是 ``PropertyChangeEvent``，其监听属性数量变化。
当多个元素被移除，调用 ``clear()`` 方法，会触发 ``ObservableList.ElementClearedEvent`` 类型事件，它将保留删除的所有元素。

.. code-block:: groovy

	def event
	def listener = {
	    if (it instanceof ObservableList.ElementEvent)  {
	        event = it
	    }
	} as PropertyChangeListener


	def observable = [1, 2, 3] as ObservableList
	observable.addPropertyChangeListener(listener)

	observable.clear()

	assert event instanceof ObservableList.ElementClearedEvent

	def elementClearedEvent = event as ObservableList.ElementClearedEvent
	assert elementClearedEvent.values == [1, 2, 3]
	assert observable.size() == 0


``ObservableMap`` and ``ObservableSet`` 概念与本章节 ``ObservableList`` 一样。


