XML 处理
============

1. Parsing XML

1.1. XmlParser and XmlSlurper

The most commonly used approach for parsing XML with Groovy is to use one of:

groovy.util.XmlParser

groovy.util.XmlSlurper

Both have the same approach to parse an xml. Both come with a bunch of overloaded parse methods plus some special methods such as parseText, parseFile and others. For the next example we will use the parseText method. It parses a XML String and recursively converts it to a list or map of objects.

XmlSlurper
def text = '''
    <list>
        <technology>
            <name>Groovy</name>
        </technology>
    </list>
'''

def list = new XmlSlurper().parseText(text) 

assert list instanceof groovy.util.slurpersupport.GPathResult 
assert list.technology.name == 'Groovy' 
Parsing the XML an returning the root node as a GPathResult
Checking we’re using a GPathResult
Traversing the tree in a GPath style
XmlParser
def text = '''
    <list>
        <technology>
            <name>Groovy</name>
        </technology>
    </list>
'''

def list = new XmlParser().parseText(text) 

assert list instanceof groovy.util.Node 
assert list.technology.name.text() == 'Groovy' 
Parsing the XML an returning the root node as a Node
Checking we’re using a Node
Traversing the tree in a GPath style
Let’s see the similarities between XMLParser and XMLSlurper first:

Both are based on SAX so they both are low memory footprint

Both can update/transform the XML

But they have key differences:

XmlSlurper evaluates the structure lazily. So if you update the xml you’ll have to evaluate the whole tree again.

XmlSlurper returns GPathResult instances when parsing XML

XmlParser returns Node objects when parsing XML

When to use one or the another?

There is a discussion at StackOverflow. The conclusions written here are based partially on this entry.
If you want to transform an existing document to another then XmlSlurper will be the choice

If you want to update and read at the same time then XmlParser is the choice.

The rationale behind this is that every time you create a node with XmlSlurper it won’t be available until you parse the document again with another XmlSlurper instance. Need to read just a few nodes XmlSlurper is for you ".

If you just have to read a few nodes XmlSlurper should be your choice, since it will not have to create a complete structure in memory"

In general both classes perform similar way. Even the way of using GPath expressions with them are the same (both use breadthFirst() and depthFirst() expressions). So I guess it depends on the write/read frequency.

1.2. DOMCategory

There is another way of parsing XML documents with Groovy with the used of groovy.xml.dom.DOMCategory which is a category class which adds GPath style operations to Java’s DOM classes.

Java has in-built support for DOM processing of XML using classes representing the various parts of XML documents, e.g. Document, Element, NodeList, Attr etc. For more information about these classes, refer to the respective JavaDocs.
Having a XML like the following:

  static def CAR_RECORDS = '''
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
  </records>
'''
You can parse it using 'groovy.xml.DOMBuilder` and groovy.xml.dom.DOMCategory.

def reader = new StringReader(CAR_RECORDS)
def doc = DOMBuilder.parse(reader) 
def records = doc.documentElement

use(DOMCategory) { 
    assert records.car.size() == 3
}
Parsing the XML
Creating DOMCategory scope to be able to use helper method calls
2. GPath

The most common way of querying XML in Groovy is using GPath:

GPath is a path expression language integrated into Groovy which allows parts of nested structured data to be identified. In this sense, it has similar aims and scope as XPath does for XML. The two main places where you use GPath expressions is when dealing with nested POJOs or when dealing with XML

It is similar to XPath expressions and you can use it not only with XML but also with POJO classes. As an example, you can specify a path to an object or element of interest:

a.b.c → for XML, yields all the <c> elements inside <b> inside <a>

a.b.c → all POJOs, yields the <c> properties for all the <b> properties of <a> (sort of like a.getB().getC() in JavaBeans)

For XML, you can also specify attributes, e.g.:

a["@href"] → the href attribute of all the a elements

a.'@href' → an alternative way of expressing this

a.@href → an alternative way of expressing this when using XmlSlurper

Let’s illustrate this with an example:

static final String books = '''
    <response version-api="2.0">
        <value>
            <books>
                <book available="20" id="1">
                    <title>Don Xijote</title>
                    <author id="1">Manuel De Cervantes</author>
                </book>
                <book available="14" id="2">
                    <title>Catcher in the Rye</title>
                   <author id="2">JD Salinger</author>
               </book>
               <book available="13" id="3">
                   <title>Alice in Wonderland</title>
                   <author id="3">Lewis Carroll</author>
               </book>
               <book available="5" id="4">
                   <title>Don Xijote</title>
                   <author id="4">Manuel De Cervantes</author>
               </book>
           </books>
       </value>
    </response>
'''
2.1. Simply traversing the tree

First thing we could do is to get a value using POJO’s notation. Let’s get the first book’s author’s name

Getting node value
def response = new XmlSlurper().parseText(books)
def authorResult = response.value.books.book[0].author

assert authorResult.text() == 'Manuel De Cervantes'
First we parse the document with XmlSlurper and the we have to consider the returning value as the root of the XML document, so in this case is "response".

That’s why we start traversing the document from response and then value.books.book[0].author. Note that in XPath the node arrays starts in [1] instead of [0], but because GPath is Java-based it begins at index 0.

In the end we’ll have the instance of the author node and because we wanted the text inside that node we should be calling the text() method. The author node is an instance of GPathResult type and text() a method giving us the content of that node as a String.

When using GPath with an xml parsed with XmlSlurper we’ll have as a result a GPathResult object. GPathResult has many other convenient methods to convert the text inside a node to any other type such as:

toInteger()

toFloat()

toBigInteger()

…​

All these methods try to convert a String to the appropriate type.

If we were using a XML parsed with XmlParser we could be dealing with instances of type Node. But still all the actions applied to GPathResult in these examples could be applied to a Node as well. Creators of both parsers took into account GPath compatibility.

Next step is to get the some values from a given node’s attribute. In the following sample we want to get the first book’s author’s id. We’ll be using two different approaches. Let’s see the code first:

Getting an attribute’s value
def response = new XmlSlurper().parseText(books)

def book = response.value.books.book[0] 
def bookAuthorId1 = book.@id 
def bookAuthorId2 = book['@id'] 

assert bookAuthorId1 == '1' 
assert bookAuthorId1.toInteger() == 1 
assert bookAuthorId1 == bookAuthorId2
Getting the first book node
Getting the book’s id attribute @id
Getting the book’s id attribute with map notation ['@id']
Getting the value as a String
Getting the value of the attribute as an Integer
As you can see there are to types of notations to get attributes, the

direct notation with @nameoftheattribute

map notation using ['@nameoftheattribute']

Both of them are equally valid.

2.2. Speed things up with breadthFirst and depthFirst

If you ever have used XPath you may have used expressions like

// : Look everywhere

/following-sibling::othernode : Look for a node "othernode" in the same level

More or less we have their counterparts in GPath with the methods breadthFirst() and depthFirst().

The first example shows a simple use of breadthFirst(). The creators of this methods created a shorter syntax for it using the symbol *.

breadthFirst()
def response = new XmlSlurper().parseText(books)

def catcherInTheRye = response.value.books.'*'.find { node->
 /* node.@id == 2 could be expressed as node['@id'] == 2 */
    node.name() == 'book' && node.@id == '2'
}

assert catcherInTheRye.title.text() == 'Catcher in the Rye'
This test searches for any node at the same level of the "books" node first, and only if it couldn’t find the node we were looking for, then it will look deeper in the tree, always taking into account the given the expression inside the closure.

The expression says Look for any node with a tag name equals 'book' having an id with a value of '2'.

But what if we would like to look for a given value without having to know exactly where it is. Let’s say that the only thing we know is the id of the author "Lewis Carroll" . How are we going to be able to find that book? depthFirst() is the solution:

depthFirst()
def response = new XmlSlurper().parseText(books)

def bookId = response.'**'.find { book->
    book.author.text() == 'Lewis Carroll'
}.@id

assert bookId == 3
depthFirst() is the same as looking something everywhere in the tree from this point down. In this case we’ve used the method find(Closure cl) to find just the first occurrence.

What if we want to collect all book’s titles?

depthFirst()
def response = new XmlSlurper().parseText(books)

def titles = response.'**'.findAll{ node-> node.name() == 'title' }*.text()

assert titles.size() == 4
It is worth mentioning again that there are some useful methods converting a node’s value to an integer, float…​ etc. Those methods could be convenient when doing comparisons like this:

helpers
def response = new XmlSlurper().parseText(books)

def titles = response.value.books.book.findAll{book->
 /* You can use toInteger() over the GPathResult object */
    book.@id.toInteger() > 2
}*.title

assert titles.size() == 2
In this case the number 2 has been hardcoded but imagine that value could have come from any other source (database…​ etc.).

3. Creating XML

The most commonly used approach for creating XML with Groovy is to use a builder, i.e. one of:

groovy.xml.MarkupBuilder

groovy.xml.StreamingMarkupBuilder

3.1. MarkupBuilder

Here is an example of using Groovy’s MarkupBuilder to create a new XML file:

Creating Xml with MarkupBuilder
def writer = new StringWriter()
def xml = new MarkupBuilder(writer) 

xml.records() { 
    car(name:'HSV Maloo', make:'Holden', year:2006) {
        country('Australia')
        record(type:'speed', 'Production Pickup Truck with speed of 271kph')
    }
    car(name:'Royale', make:'Bugatti', year:1931) {
        country('France')
        record(type:'price', 'Most Valuable Car at $15 million')
    }
}

def records = new XmlSlurper().parseText(writer.toString()) 

assert records.car.first().name.text() == 'HSV Maloo'
assert records.car.last().name.text() == 'Royale'
Create an instance of MarkupBuilder
Start creating the XML tree
Create an instance of XmlSlurper to traverse and test the generated XML
Let’s take a look a little bit closer:

Creating XML elements
def xmlString = "<movie>the godfather</movie>" 

def xmlWriter = new StringWriter() 
def xmlMarkup = new MarkupBuilder(xmlWriter)

xmlMarkup.movie("the godfather") 

assert xmlString == xmlWriter.toString() 
We’re creating a reference string to compare against
The xmlWriter instance is used by MarkupBuilder to convert the xml representation to a String instance eventually
The xmlMarkup.movie(…​) call will create a XML node with a tag called movie and with content the godfather.
Creating XML elements with attributes
def xmlString = "<movie id='2'>the godfather</movie>"

def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter)

xmlMarkup.movie(id: "2", "the godfather") 

assert xmlString == xmlWriter.toString()
This time in order to create both attributes and node content you can create as many map entries as you like and finally add a value to set the node’s content
The value could be any Object, the value will be serialized to its String representation.
Creating XML nested elements
def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter)

xmlMarkup.movie(id: 2) { 
    name("the godfather")
}

def movie = new XmlSlurper().parseText(xmlWriter.toString())

assert movie.@id == 2
assert movie.name.text() == 'the godfather'
A closure represents the children elements of a given node. Notice this time instead of using a String for the attribute we’re using a number.
Sometimes you may want to use a specific namespace in your xml documents:

Namespace aware
def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter)

xmlMarkup
    .'x:movies'('xmlns:x':'http://www.groovy-lang.org') { 
        'x:movie'(id: 1, 'the godfather')
        'x:movie'(id: 2, 'ronin')
    }

def movies =
    new XmlSlurper() 
        .parseText(xmlWriter.toString())
        .declareNamespace(x:'http://www.groovy-lang.org')

assert movies.'x:movie'.last().@id == 2
assert movies.'x:movie'.last().text() == 'ronin'
Creating a node with a given namespace xmlns:x
Creating a XmlSlurper registering the namespace to be able to test the XML we just created
What about having some more meaningful example. We may want to generate more elements, to have some logic when creating our XML:

Mix code
def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter)

xmlMarkup
    .'x:movies'('xmlns:x':'http://www.groovy-lang.org') {
        (1..3).each { n -> 
            'x:movie'(id: n, "the godfather $n")
            if (n % 2 == 0) { 
                'x:movie'(id: n, "the godfather $n (Extended)")
            }
        }
    }

def movies =
    new XmlSlurper()
        .parseText(xmlWriter.toString())
        .declareNamespace(x:'http://www.groovy-lang.org')

assert movies.'x:movie'.size() == 4
assert movies.'x:movie'*.text().every { name -> name.startsWith('the')}
Generating elements from a range
Using a conditional for creating a given element
Of course the instance of a builder can be passed as a parameter to refactor/modularize your code:

Mix code
def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter)


Closure<MarkupBuilder> buildMovieList = { MarkupBuilder builder ->
    (1..3).each { n ->
        builder.'x:movie'(id: n, "the godfather $n")
        if (n % 2 == 0) {
            builder.'x:movie'(id: n, "the godfather $n (Extended)")
        }
    }

    return builder
}

xmlMarkup.'x:movies'('xmlns:x':'http://www.groovy-lang.org') {
    buildMovieList(xmlMarkup) 
}

def movies =
    new XmlSlurper()
        .parseText(xmlWriter.toString())
        .declareNamespace(x:'http://www.groovy-lang.org')

assert movies.'x:movie'.size() == 4
assert movies.'x:movie'*.text().every { name -> name.startsWith('the')}
In this case we’ve created a Closure to handle the creation of a list of movies
Just using the buildMovieList function when necessary
3.2. StreamingMarkupBuilder

The class groovy.xml.StreamingMarkupBuilder is a builder class for creating XML markup. This implementation uses a groovy.xml.streamingmarkupsupport.StreamingMarkupWriter to handle output.

Using StreamingMarkupBuilder
def xml = new StreamingMarkupBuilder().bind { 
    records {
        car(name:'HSV Maloo', make:'Holden', year:2006) { 
            country('Australia')
            record(type:'speed', 'Production Pickup Truck with speed of 271kph')
        }
        car(name:'P50', make:'Peel', year:1962) {
            country('Isle of Man')
            record(type:'size', 'Smallest Street-Legal Car at 99cm wide and 59 kg in weight')
        }
        car(name:'Royale', make:'Bugatti', year:1931) {
            country('France')
            record(type:'price', 'Most Valuable Car at $15 million')
        }
    }
}

def records = new XmlSlurper().parseText(xml.toString()) 

assert records.car.size() == 3
assert records.car.find { it.@name == 'P50' }.country.text() == 'Isle of Man'
Note that StreamingMarkupBuilder.bind returns a Writable instance that may be used to stream the markup to a Writer
We’re capturing the output in a String to parse it again an check the structure of the generated XML with XmlSlurper.
3.3. MarkupBuilderHelper

The groovy.xml.MarkupBuilderHelper is, as its name reflects, a helper for groovy.xml.MarkupBuilder.

This helper normally can be accessed from within an instance of class groovy.xml.MarkupBuilder or an instance of groovy.xml.StreamingMarkupBuilder.

This helper could be handy in situations when you may want to:

Produce a comment in the output

Produce an XML processing instruction in the output

Produce an XML declaration in the output

Print data in the body of the current tag, escaping XML entities

Print data in the body of the current tag

In both MarkupBuilder and StreamingMarkupBuilder this helper is accessed by the property mkp:

Using MarkupBuilder’s 'mkp'
def xmlWriter = new StringWriter()
def xmlMarkup = new MarkupBuilder(xmlWriter).rules {
    mkp.comment('THIS IS THE MAIN RULE') 
    rule(sentence: mkp.yield('3 > n')) 
}


assert xmlWriter.toString().contains('3 &gt; n')
assert xmlWriter.toString().contains('<!-- THIS IS THE MAIN RULE -->')
Using mkp to create a comment in the XML
Using mkp to generate an escaped value
Checking both assumptions were true
Here is another example to show the use of mkp property accessible from within the bind method scope when using StreamingMarkupBuilder:

Using StreamingMarkupBuilder’s 'mkp'
def xml = new StreamingMarkupBuilder().bind {
    records {
        car(name: mkp.yield('3 < 5')) 
        car(name: mkp.yieldUnescaped('1 < 3')) 
    }
}

assert xml.toString().contains('3 &lt; 5')
assert xml.toString().contains('1 < 3')
If we want to generate a escaped value for the name attribute with mkp.yield
Checking the values later on with XmlSlurper
3.4. DOMToGroovy

Suppose we have an existing XML document and we want to automate generation of the markup without having to type it all in? We just need to use org.codehaus.groovy.tools.xml.DOMToGroovy as shown in the following example:

Building MarkupBuilder from DOMToGroovy
def songs  = """
    <songs>
      <song>
        <title>Here I go</title>
        <band>Whitesnake</band>
      </song>
    </songs>
"""

def builder     =
    javax.xml.parsers.DocumentBuilderFactory.newInstance().newDocumentBuilder()

    def inputStream = new ByteArrayInputStream(songs.bytes)
    def document    = builder.parse(inputStream)
    def output      = new StringWriter()
    def converter   = new DomToGroovy(new PrintWriter(output)) 

    converter.print(document) 

    String xmlRecovered  =
        new GroovyShell()
        .evaluate("""
           def writer = new StringWriter()
           def builder = new groovy.xml.MarkupBuilder(writer)
           builder.${output}

           return writer.toString()
        """) 

    assert new XmlSlurper().parseText(xmlRecovered).song.title.text() == 'Here I go' 
Creating DOMToGroovy instance
Converts the XML to MarkupBuilder calls which are available in the output StringWriter
Using output variable to create the whole MarkupBuilder
Back to XML string
4. Manipulating XML

In this chapter you’ll see the different ways of adding / modifying / removing nodes using XmlSlurper or XmlParser. The xml we are going to be handling is the following:

def xml = """
<response version-api="2.0">
    <value>
        <books>
            <book id="2">
                <title>Don Xijote</title>
                <author id="1">Manuel De Cervantes</author>
            </book>
        </books>
    </value>
</response>
"""
4.1. Adding nodes

The main difference between XmlSlurper and XmlParser is that when former creates the nodes they won’t be available until the document’s been evaluated again, so you should parse the transformed document again in order to be able to see the new nodes. So keep that in mind when choosing any of both approaches.

If you needed to see a node right after creating it then XmlParser should be your choice, but if you’re planning to do many changes to the XML and send the result to another process maybe XmlSlurper would be more efficient.

You can’t create a new node directly using the XmlSlurper instance, but you can with XmlParser. The way of creating a new node from XmlParser is through its method createNode(..)

def parser = new XmlParser()
def response = parser.parseText(xml)
def numberOfResults = parser.createNode(
    response,
    new QName("numberOfResults"),
    [:]
)

numberOfResults.value = "1"
assert response.numberOfResults.text() == "1"
The createNode() method receives the following parameters:

parent node (could be null)

The qualified name for the tag (In this case we only use the local part without any namespace). We’re using an instance of groovy.xml.QName

A map with the tag’s attributes (None in this particular case)

Anyway you won’t normally be creating a node from the parser instance but from the parsed XML instance. That is from a Node or a GPathResult instance.

Take a look at the next example. We are parsing the xml with XmlParser and then creating a new node from the parsed document’s instance (Notice the method here is slightly different in the way it receives the parameters):

def parser = new XmlParser()
def response = parser.parseText(xml)

response.appendNode(
    new QName("numberOfResults"),
    [:],
    "1"
)

response.numberOfResults.text() == "1"
When using XmlSlurper, GPathResult instances don’t have createNode() method.

4.2. Modifying / Removing nodes

We know how to parse the document, add new nodes, now I want to change a given node’s content. Let’s start using XmlParser and Node. This example changes the first book information to actually another book.

   def response = new XmlParser().parseText(xml)

/* Use the same syntax as groovy.xml.MarkupBuilder */
   response.value.books.book[0].replaceNode{ 
       book(id:"3"){
           title("To Kill a Mockingbird")
           author(id:"3","Harper Lee")
       }
   }

   def newNode = response.value.books.book[0]

   assert newNode.name() == "book"
   assert newNode.@id == "3"
   assert newNode.title.text() == "To Kill a Mockingbird"
   assert newNode.author.text() == "Harper Lee"
   assert newNode.author.@id.first() == "3"
When using replaceNode() the closure we pass as parameter should follow the same rules as if we were using groovy.xml.MarkupBuilder:

Here’s the same example using XmlSlurper:

def response = new XmlSlurper().parseText(books)

/* Use the same syntax as groovy.xml.MarkupBuilder */
response.value.books.book[0].replaceNode{
    book(id:"3"){
        title("To Kill a Mockingbird")
        author(id:"3","Harper Lee")
    }
}

assert response.value.books.book[0].title.text() == "Don Xijote"

/* That mkp is a special namespace used to escape away from the normal building mode
   of the builder and get access to helper markup methods
   'yield', 'pi', 'comment', 'out', 'namespaces', 'xmlDeclaration' and
   'yieldUnescaped' */
def result = new StreamingMarkupBuilder().bind { mkp.yield response }.toString()
def changedResponse = new XmlSlurper().parseText(result)

assert changedResponse.value.books.book[0].title.text() == "To Kill a Mockingbird"
Notice how using XmlSlurper we have to parse the transformed document again in order to find the created nodes. In this particular example could be a little bit annoying isn’t it?

Finally both parsers also use the same approach for adding a new attribute to a given attribute. This time again the difference is whether you want the new nodes to be available right away or not. First XmlParser:

def parser = new XmlParser()
def response = parser.parseText(xml)

response.@numberOfResults = "1"

assert response.@numberOfResults == "1"
And XmlSlurper:

def response = new XmlSlurper().parseText(books)
response.@numberOfResults = "2"

assert response.@numberOfResults == "2"
When using XmlSlurper, adding a new attribute does not require you to perform a new evaluation.

4.3. Printing XML

4.3.1. XmlUtil

Sometimes is useful to get not only the value of a given node but the node itself (for instance to add this node to another XML).

For that you can use groovy.xml.XmlUtil class. It has several static methods to serialize the xml fragment from several type of sources (Node, GPathResult, String…​)

Getting a node as a string
def response = new XmlParser().parseText(xml)
def nodeToSerialize = response.'**'.find {it.name() == 'author'}
def nodeAsText = XmlUtil.serialize(nodeToSerialize)

assert nodeAsText ==
    XmlUtil.serialize('<?xml version="1.0" encoding="UTF-8"?><author id="1">Manuel De Cervantes</author>')