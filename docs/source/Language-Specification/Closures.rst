闭包
======


This chapter covers Groovy Closures. A closure in Groovy is an open, anonymous, block of code that can take arguments, return a value and be assigned to a variable. A closure may reference variables declared in its surrounding scope. In opposition to the formal definition of a closure, Closure in the Groovy language can also contain free variables which are defined outside of its surrounding scope. While breaking the formal concept of a closure, it offers a variety of advantages which are described in this chapter.

1. Syntax

1.1. Defining a closure

A closure definition follows this syntax:

{ [closureParameters -> ] statements }
Where [closureParameters->] is an optional comma-delimited list of parameters, and statements are 0 or more Groovy statements. The parameters look similar to a method parameter list, and these parameters may be typed or untyped.

When a parameter list is specified, the -> character is required and serves to separate the arguments from the closure body. The statements portion consists of 0, 1, or many Groovy statements.

Some examples of valid closure definitions:

.. code-block:: groovy


    { item++ }                                          

    { -> item++ }                                       

    { println it }                                      

    { it -> println it }                                

    { name -> println name }                            

    { String x, int y ->                                
        println "hey ${x} the value is ${y}"
    }

    { reader ->                                         
        def line = reader.readLine()
        line.trim()
    }

A closure referencing a variable named item
It is possible to explicitly separate closure parameters from code by adding an arrow (->)
A closure using an implicit parameter (it)
An alternative version where it is an explicit parameter
In that case it is often better to use an explicit name for the parameter
A closure accepting two typed parameters
A closure can contain multiple statements
1.2. Closures as an object

A closure is an instance of the groovy.lang.Closure class, making it assignable to a variable or a field as any other variable, despite being a block of code:

def listener = { e -> println "Clicked on $e.source" }      
assert listener instanceof Closure
Closure callback = { println 'Done!' }                      
Closure<Boolean> isTextFile = {
    File it -> it.name.endsWith('.txt')                     
}
You can assign a closure to a variable, and it is an instance of groovy.lang.Closure
If not using def, you can assign a closure to a variable of type groovy.lang.Closure
Optionally, you can specify the return type of the closure by using the generic type of groovy.lang.Closure
1.3. Calling a closure

A closure, as an anonymous block of code, can be called like any other method. If you define a closure which takes no argument like this:

def code = { 123 }
Then the code inside the closure will only be executed when you call the closure, which can be done by using the variable as if it was a regular method:

assert code() == 123
Alternatively, you can be explicit and use the call method:

assert code.call() == 123
The principle is the same if the closure accepts arguments:

def isOdd = { int i-> i%2 == 1 }                            
assert isOdd(3) == true                                     
assert isOdd.call(2) == false                               

def isEven = { it%2 == 0 }                                  
assert isEven(3) == false                                   
assert isEven.call(2) == true                               
define a closure which accepts an int as a parameter
it can be called directly
or using the call method
same goes for a closure with an implicit argument (it)
which can be called directly using (arg)
or using call
Unlike a method, a closure always returns a value when called. The next section discusses how to declare closure arguments, when to use them and what is the implicit "it" parameter.

2. Parameters

2.1. Normal parameters

Parameters of closures follow the same principle as parameters of regular methods:

an optional type

a name

an optional default value

Parameters are separated with commas:

def closureWithOneArg = { str -> str.toUpperCase() }
assert closureWithOneArg('groovy') == 'GROOVY'

def closureWithOneArgAndExplicitType = { String str -> str.toUpperCase() }
assert closureWithOneArgAndExplicitType('groovy') == 'GROOVY'

def closureWithTwoArgs = { a,b -> a+b }
assert closureWithTwoArgs(1,2) == 3

def closureWithTwoArgsAndExplicitTypes = { int a, int b -> a+b }
assert closureWithTwoArgsAndExplicitTypes(1,2) == 3

def closureWithTwoArgsAndOptionalTypes = { a, int b -> a+b }
assert closureWithTwoArgsAndOptionalTypes(1,2) == 3

def closureWithTwoArgAndDefaultValue = { int a, int b=2 -> a+b }
assert closureWithTwoArgAndDefaultValue(1) == 3
2.2. Implicit parameter

When a closure does not explicitly define a parameter list (using ->), a closure always defines an implicit parameter, named it. This means that this code:

def greeting = { "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
is stricly equivalent to this one:

def greeting = { it -> "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
If you want to declare a closure which accepts no argument and must be restricted to calls without arguments, then you must declare it with an explicit empty argument list:

def magicNumber = { -> 42 }

// this call will fail because the closure doesn't accept any argument
magicNumber(11)
2.3. Varargs

It is possible for a closure to declare variable arguments like any other method. Vargs methods are methods that can accept a variable number of arguments if the last parameter is of variable length (or an array) like in the next examples:

def concat1 = { String... args -> args.join('') }           
assert concat1('abc','def') == 'abcdef'                     
def concat2 = { String[] args -> args.join('') }            
assert concat2('abc', 'def') == 'abcdef'

def multiConcat = { int n, String... args ->                
    args.join('')*n
}
assert multiConcat(2, 'abc','def') == 'abcdefabcdef'
A closure accepting a variable number of strings as first parameter
It may be called using any number of arguments without having to explicitly wrap them into an array
The same behavior is directly available if the args parameter is declared as an array
As long as the last parameter is an array of an explicit vargs type
3. Delegation strategy

3.1. Groovy closures vs lambda expressions

Groovy defines closures as instances of the Closure class. It makes it very different from lambda expressions in Java 8. Delegation is a key concept in Groovy closures which has no equivalent in lambdas. The ability to change the delegate or change the delegation strategy of closures make it possible to design beautiful domain specific languages (DSLs) in Groovy.

3.2. Owner, delegate and this

To understand the concept of delegate, we must first explain the meaning of this inside a closure. A closure actually defines 3 distinct things:

this corresponds to the enclosing class where the closure is defined

owner corresponds to the enclosing object where the closure is defined, which may be either a class or a closure

delegate corresponds to a third party object where methods calls or properties are resolved whenever the receiver of the message is not defined

3.2.1. The meaning of this

In a closure, calling getThisObject will return the enclosing class where the closure is defined. It is equivalent to using an explicit this:

class Enclosing {
    void run() {
        def whatIsThisObject = { getThisObject() }          
        assert whatIsThisObject() == this                   
        def whatIsThis = { this }                           
        assert whatIsThis() == this                         
    }
}
class EnclosedInInnerClass {
    class Inner {
        Closure cl = { this }                               
    }
    void run() {
        def inner = new Inner()
        assert inner.cl() == inner                          
    }
}
class NestedClosures {
    void run() {
        def nestedClosures = {
            def cl = { this }                               
            cl()
        }
        assert nestedClosures() == this                     
    }
}
a closure is defined inside the Enclosing class, and returns getThisObject
calling the closure will return the instance of Enclosing where the the closure is defined
in general, you will just want to use the shortcut this notation
and it returns exactly the same object
if the closure is defined in a inner class
this in the closure will return the inner class, not the top-level one
in case of nested closures, like here cl being defined inside the scope of nestedClosures
then this corresponds to the closest outer class, not the enclosing closure!
It is of course possible to call methods from the enclosing class this way:

class Person {
    String name
    int age
    String toString() { "$name is $age years old" }

    String dump() {
        def cl = {
            String msg = this.toString()               
            println msg
            msg
        }
        cl()
    }
}
def p = new Person(name:'Janice', age:74)
assert p.dump() == 'Janice is 74 years old'
the closure calls toString on this, which will actually call the toString method on the enclosing object, that is to say the Person instance
3.2.2. Owner of a closure

The owner of a closure is very similar to the definition of this in a closure with a subtle difference: it will return the direct enclosing object, be it a closure or a class:

class Enclosing {
    void run() {
        def whatIsOwnerMethod = { getOwner() }               
        assert whatIsOwnerMethod() == this                   
        def whatIsOwner = { owner }                          
        assert whatIsOwner() == this                         
    }
}
class EnclosedInInnerClass {
    class Inner {
        Closure cl = { owner }                               
    }
    void run() {
        def inner = new Inner()
        assert inner.cl() == inner                           
    }
}
class NestedClosures {
    void run() {
        def nestedClosures = {
            def cl = { owner }                               
            cl()
        }
        assert nestedClosures() == nestedClosures            
    }
}
a closure is defined inside the Enclosing class, and returns getOwner
calling the closure will return the instance of Enclosing where the the closure is defined
in general, you will just want to use the shortcut owner notation
and it returns exactly the same object
if the closure is defined in a inner class
owner in the closure will return the inner class, not the top-level one
but in case of nested closures, like here cl being defined inside the scope of nestedClosures
then owner corresponds to the enclosing closure, hence a different object from this!
3.2.3. Delegate of a closure

The delegate of a closure can be accessed by using the delegate property or calling the getDelegate method. It is a powerful concept for building domain specific languages in Groovy. While closure-this and closure-owner refer to the lexical scope of a closure, the delegate is a user defined object that a closure will use. By default, the delegate is set to owner:

class Enclosing {
    void run() {
        def cl = { getDelegate() }                          
        def cl2 = { delegate }                              
        assert cl() == cl2()                                
        assert cl() == this                                 
        def enclosed = {
            { -> delegate }.call()                          
        }
        assert enclosed() == enclosed                       
    }
}
you can get the delegate of a closure calling the getDelegate method
or using the delegate property
both return the same object
which is the enclosing class or closure
in particular in case of nested closures
delegate will correspond to the owner
The delegate of a closure can be changed to any object. Let’s illustrate this by creating two classes which are not subclasses of each other but both define a property called name:

class Person {
    String name
}
class Thing {
    String name
}

def p = new Person(name: 'Norman')
def t = new Thing(name: 'Teapot')
Then let’s define a closure which fetches the name property on the delegate:

def upperCasedName = { delegate.name.toUpperCase() }
Then by changing the delegate of the closure, you can see that the target object will change:

upperCasedName.delegate = p
assert upperCasedName() == 'NORMAN'
upperCasedName.delegate = t
assert upperCasedName() == 'TEAPOT'
At this point, the behavior is not different from having a `variable defined in the lexical scope of the closure:

def target = p
def upperCasedNameUsingVar = { target.name.toUpperCase() }
assert upperCasedNameUsingVar() == 'NORMAN'
However there is are major differences:

in the last example, target is a local variable referenced from within the closure

the delegate can be used transparently, that is to say without prefixing method calls with delegate. as explained in the next paragraph.

3.2.4. Delegation strategy

Whenever, in a closure, a property is accessed without explicitly setting a receiver object, then a delegation strategy is involved:

class Person {
    String name
}
def p = new Person(name:'Igor')
def cl = { name.toUpperCase() }                 
cl.delegate = p                                 
assert cl() == 'IGOR'                           
name is not referencing a variable in the lexical scope of the closure
we can change the delegate of the closure to be an instance of Person
and the method call will succeed
The reason this code works is that the name property will be resolved transparently on the delegate object! This is a very powerful way to resolve properties or method calls inside closures. There’s no need to set an explicit delegate. receiver: the call will be made because the default delegation strategy of the closure makes it so. A closure actually defines multiple resolution strategies that you can choose:

Closure.OWNER_FIRST is the default strategy. If a property/method exists on the owner, then it will be called on the owner. If not, then the delegate is used.

Closure.DELEGATE_FIRST reverses the logic: the delegate is used first, then the owner

Closure.OWNER_ONLY will only resolve the property/method lookup on the owner: the delegate will be ignored.

Closure.DELEGATE_ONLY will only resolve the property/method lookup on the delegate: the owner will be ignored.

Closure.TO_SELF can be used by developers who need advanced meta-programming techniques and wish to implement a custom resolution strategy: the resolution will not be made on the owner or the delegate but only on the closure class itself. It makes only sense to use this if you implement your own subclass of Closure.

Let’s illustrate the default "owner first" strategy with this code:

class Person {
    String name
    def pretty = { "My name is $name" }             
    String toString() {
        pretty()
    }
}
class Thing {
    String name                                     
}

def p = new Person(name: 'Sarah')
def t = new Thing(name: 'Teapot')

assert p.toString() == 'My name is Sarah'           
p.pretty.delegate = t                               
assert p.toString() == 'My name is Sarah'           
for the illustration, we define a closure member which references "name"
both the Person and the Thing class define a name property
Using the default strategy, the name property is resolved on the owner first
so if we change the delegate to t which is an instance of Thing
there is no change in the result: name is first resolved on the owner of the closure
However, it is possible to change the resolution strategy of the closure:

p.pretty.resolveStrategy = Closure.DELEGATE_FIRST
assert p.toString() == 'My name is Teapot'
By changing the resolveStrategy, we are modifying the way Groovy will resolve the "implicit this" references: in this case, name will first be looked in the delegate, then if not found, on the owner. Since name is defined in the delegate, an instance of Thing, then this value is used.

The difference between "delegate first" and "delegate only" or "owner first" and "owner only" can be illustrated if one of the delegate (resp. owner) does not have such a method or property:

class Person {
    String name
    int age
    def fetchAge = { age }
}
class Thing {
    String name
}

def p = new Person(name:'Jessica', age:42)
def t = new Thing(name:'Printer')
def cl = p.fetchAge
cl.delegate = p
assert cl() == 42
cl.delegate = t
assert cl() == 42
cl.resolveStrategy = Closure.DELEGATE_ONLY
cl.delegate = p
assert cl() == 42
cl.delegate = t
try {
cl()
    assert false
} catch (MissingPropertyException ex) {
    // "age" is not defined on the delegate
}
In this example, we define two classes which both have a name property but only the Person class declares an age. The Person class also declares a closure which references age. We can change the default resolution strategy from "owner first" to "delegate only". Since the owner of the closure is the Person class, then we can check that if the delegate is an instance of Person, calling the closure is successful, but if we call it with a delegate being an instance of Thing, it fails with a groovy.lang.MissingPropertyException. Despite the closure being defined inside the Person class, the owner is not used.

A comprehensive explanation about how to use this feature to develop DSLs can be found in a dedicated section of the manual.
4. Closures in GStrings

Take the following code:

def x = 1
def gs = "x = ${x}"
assert gs == 'x = 1'
The code behaves as you would expect, but what happens if you add:

x = 2
assert gs == 'x = 2'
You will see that the assert fails! There are two reasons for this:

a GString only evaluates lazily the toString representation of values

the syntax ${x} in a GString does not represent a closure but an expression to $x, evaluated when the GString is created.

In our example, the GString is created with an expression referencing x. When the GString is created, the value of x is 1, so the GString is created with a value of 1. When the assert is triggered, the GString is evaluated and 1 is converted to a String using toString. When we change x to 2, we did change the value of x, but it is a different object, and the GString still references the old one.

A GString will only change its toString representation if the values it references are mutating. If the references change, nothing will happen.
If you need a real closure in a GString and for example enforce lazy evaluation of variables, you need to use the alternate syntax ${→ x} like in the fixed example:

def x = 1
def gs = "x = ${-> x}"
assert gs == 'x = 1'

x = 2
assert gs == 'x = 2'
And let’s illustrate how it differs from mutation with this code:

class Person {
    String name
    String toString() { name }          
}
def sam = new Person(name:'Sam')        
def lucy = new Person(name:'Lucy')      
def p = sam                             
def gs = "Name: ${p}"                   
assert gs == 'Name: Sam'                
p = lucy                                
assert gs == 'Name: Sam'                
sam.name = 'Lucy'                       
assert gs == 'Name: Lucy'               
the Person class has a toString method returning the name property
we create a first Person named Sam
we create another Person named Lucy
the p variable is set to Sam
and a closure is created, referencing the value of p, that is to say Sam
so when we evaluate the string, it returns Sam
if we change p to Lucy
the string still evaluates to Sam because it was the value of p when the GString was created
so if we mutate Sam to change his name to Lucy
this time the GString is correctly mutated
So if you don’t want to rely on mutating objects or wrapping objects, you must use closures in GString by explicitly declaring an empty argument list:

class Person {
    String name
    String toString() { name }
}
def sam = new Person(name:'Sam')
def lucy = new Person(name:'Lucy')
def p = sam
// Create a GString with lazy evaluation of "p"
def gs = "Name: ${-> p}"
assert gs == 'Name: Sam'
p = lucy
assert gs == 'Name: Lucy'
5. Closure coercion

Closures can be converted into interfaces or single-abstract method types. Please refer to this section of the manual for a complete description.

6. Functional programming

Closures, like lambda expressions in Java 8 are at the core of the functional programming paradigm in Groovy. Some functional programming operations on functions are available directly on the Closure class, like illustrated in this section.

6.1. Currying

In Groovy, currying refers to the concept of partial application. It does not correspond to the real concept of currying in functional programming because of the different scoping rules that Groovy applies on closures. Currying in Groovy will let you set the value of one parameter of a closure, and it will return a new closure accepting one less argument.

6.1.1. Left currying

Left currying is the fact of setting the left-most parameter of a closure, like in this example:

def nCopies = { int n, String str -> str*n }    
def twice = nCopies.curry(2)                    
assert twice('bla') == 'blabla'                 
assert twice('bla') == nCopies(2, 'bla')        
the nCopies closure defines two parameters
curry will set the first parameter to 2, creating a new closure (function) which accepts a single String
so the new function call be called with only a String
and it is equivalent to calling nCopies with two parameters
6.1.2. Right currying

Similarily to left currying, it is possible to set the right-most parameter of a closure:

def nCopies = { int n, String str -> str*n }    
def blah = nCopies.rcurry('bla')                
assert blah(2) == 'blabla'                      
assert blah(2) == nCopies(2, 'bla')             
the nCopies closure defines two parameters
rcurry will set the last parameter to bla, creating a new closure (function) which accepts a single int
so the new function call be called with only an int
and it is equivalent to calling nCopies with two parameters
6.1.3. Index based currying

In case a closure accepts more than 2 parameters, it is possible to set an arbitrary parameter using ncurry:

def volume = { double l, double w, double h -> l*w*h }      
def fixedWidthVolume = volume.ncurry(1, 2d)                 
assert volume(3d, 2d, 4d) == fixedWidthVolume(3d, 4d)       
def fixedWidthAndHeight = volume.ncurry(1, 2d, 4d)          
assert volume(3d, 2d, 4d) == fixedWidthAndHeight(3d)        
the volume function defines 3 parameters
ncurry will set the second parameter (index = 1) to 2d, creating a new volume function which accepts length and height
that function is equivalent to calling volume omitting the width
it is also possible to set multiple parameters, starting from the specified index
the resulting function accepts as many parameters as the initial one minus the number of parameters set by ncurry
6.2. Memoization

Memoization allows the result of the call of a closure to be cached. It is interesting if the computation done by a function (closure) is slow, but you know that this function is going to be called often with the same arguments. A typical example is the Fibonacci suite. A naive implementation may look like this:

def fib
fib = { long n -> n<2?n:fib(n-1)+fib(n-2) }
assert fib(15) == 610 // slow!
It is a naive implementation because 'fib' is often called recursively with the same arguments, leading to an exponential algorithm:

computing fib(15) requires the result of fib(14) and fib(13)

computing fib(14) requires the result of fib(13) and fib(12)

Since calls are recursive, you can already see that we will compute the same values again and again, although they could be cached. This naive implementation can be "fixed" by caching the result of calls using memoize:

fib = { long n -> n<2?n:fib(n-1)+fib(n-2) }.memoize()
assert fib(25) == 75025 // fast!
The cache works using the actual values of the arguments. This means that you should be very careful if you use memoization with something else than primitive or boxed primitive types.
The behavior of the cache can be tweaked using alternate methods:

memoizeAtMost will generate a new closure which caches at most n values

memoizeAtLeast will generate a new closure which caches at least n values

memoizeBetween will generate a new closure which caches at least n values and at most n values

The cache used in all memoize variants is a LRU cache.

6.3. Composition

Closure composition corresponds to the concept of function composition, that is to say creating a new function by composing two or more functions (chaining calls), as illustrated in this example:

def plus2  = { it + 2 }
def times3 = { it * 3 }

def times3plus2 = plus2 << times3
assert times3plus2(3) == 11
assert times3plus2(4) == plus2(times3(4))

def plus2times3 = times3 << plus2
assert plus2times3(3) == 15
assert plus2times3(5) == times3(plus2(5))

// reverse composition
assert times3plus2(3) == (times3 >> plus2)(3)
6.4. Trampoline

Recursive algorithms are often restricted by a physical limit: the maximum stack height. For example, if you call a method that recursively calls itself too deep, you will eventually receive a StackOverflowException.

An approach that helps in those situations is by using Closure and its trampoline capability.

Closures are wrapped in a TrampolineClosure. Upon calling, a trampolined Closure will call the original Closure waiting for its result. If the outcome of the call is another instance of a TrampolineClosure, created perhaps as a result to a call to the trampoline() method, the Closure will again be invoked. This repetitive invocation of returned trampolined Closures instances will continue until a value other than a trampolined Closure is returned. That value will become the final result of the trampoline. That way, calls are made serially, rather than filling the stack.

Here’s an example of the use of trampoline() to implement the factorial function:

def factorial
factorial = { int n, def accu = 1G ->
    if (n < 2) return accu
    factorial.trampoline(n - 1, n * accu)
}
factorial = factorial.trampoline()

assert factorial(1)    == 1
assert factorial(3)    == 1 * 2 * 3
assert factorial(1000) // == 402387260.. plus another 2560 digits
6.5. Method pointers

It is often practical to be able to use a regular method as a closure. For example, you might want to use the currying abilities of a closure, but those are not available to normal methods. In Groovy, you can obtain a closure from any method with the method pointer operator.