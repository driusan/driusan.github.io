---
layout: default
title: Is Go Object Oriented?
---

A question that gets asked surprisingly often by people new to Go is "is Go object oriented?",
and for a simple question that isn't about text editors or whitespace it's surprisingly controversial.

The reason is because there's really two, somewhat related, questions that the question is asking.

1. Does Go meet all the requirements to meet the technical definition of an object oriented programming language?
2. Does Go meet the expectations and assumptions of programmers coming from other object oriented languages?

The answer to the first question is "Yes, probably." The answer to the second is "No, probably not." It's somewhat unfortunate
that the answer to the first question isn't evidently "no," because then everyone would at least be on the same page.

Go is a derivative of Plan 9's C dialect. C++ is a derivate of C too, so that doesn't mean that Go can't be object oriented, but 
unlike C++, Go wasn't designed with an explicit goal *of being* an OOP language, so if the extensions that they've added 
are enough to make it technically meet the requirements of OOP, then that's a side-effect, not a goal. Looking at it this
way helps avoid some of the confusion people have when trying to answer the question "is Go object oriented?"

So does Go have all the requirements to meet the technical definition of an object oriented programming language? If
we answer this question, maybe we can answer the second, more important question, at the same time.

If I recall from way back when I was a taking CS101 and had to answer the question "What are the 3 principles of object oriented
programming?" on an exam (forgive me if I get some of this wrong, it was a long time ago..) the answer was that there
were 3 properties that it took to be object oriented:

1. Encapsulation
2. Inheritance
3. Polymorphism

Does Go meet these requirements?

1. Encapsulation is the ability to take data and methods that act on that data, and encapsulate them into an object. For OO
    purists, you should be forced to use methods to set or retrieve data, so that you can do validation or transformations to
    the internal representation of the data.

    Go has structs, and receiver functions can be added to any type. You can hide struct fields by making them non-exported
    (name starts with a lower case letter) and force people to use exported (name starts with a capital letter) receiver functions
    to access the data. Even though it's called a "struct" and not a "class", and it doesn't have the words "public" and "private," 
    I think it's fair to say Go meets this requirement. (N.B. Forcing people to use Get/Setter methods isn't idiomatic Go, but it's
    possible.)

2. Inheritance is the ability to take a class, and create another class which inherits the behaviour of that class.

    Go doesn't have an explicit "extends" keyword, but if you take a struct, and include a type as an anonymous member
    of the struct, the struct will implicitly have all the members and receivers of the anonymous type accessible through
    your struct, making it appear as if you've "inherited" from the type. This is probably easiest to show with an example

        type Foo struct {
        	Val int
        }
        
        func (f Foo) Noop() {
        	return
        }
        
        type Bar struct {
        	Foo
        }

    In this example, Foo is a struct that has an integer Val as a member and a receiver function Noop() that does nothing. Bar is another
    struct which composes in Foo as an anonymous member. You can access things that come from Foo within a Bar struct as if they belonged 
     to Bar. So, for instance, this is perfectly valid, if useless, code:

        
        var b Bar
        b.Val = 300
        b.Noop()

    Is this inheritance? It certainly looks enough like inheritance to most people that people new to Go or coming to Go from object-oriented
    languages call it that, but in the official documentation it's always referred to as "struct composition." Why? Because "inheritance" to most
    people implies a type of polymorphism that composition doesn't, and thinking of it as "inheritance" leads to confusion that we'll get into as
    we look at the third requirement..

3.  "Polymorphism is the provision of a single interface to entities of different types," according to wikipedia.

    Go acheives polymorphism via interfaces. You can declare an interface, and anything which implements the methods of
    that interface implicitly implements it. Functions can take interfaces instead of concrete types as parameters, and then
    be passed anything that implements that interface.

    This is certainly polymorphic according to the wikipedia definition, but acheived in a completely different way than people coming from 
    OOP languages expect.

     In most object oriented languages, the most common method of acheiving polymorphism is through inheritance, and by grouping classes into
     a taxonomy where children can override methods of their parents. Go does not work that way. Let's modify our example a little.

        type Foo struct {
        }
        
        func (f Foo) SomeMethod() string {
        	return "something"
        }
        func (f Foo) AnOp() string{
        	return f.SomeMethod()
        }
        
        type Bar struct {
        	Foo
        }
        
        func (b Bar) SomeMethod() string {
        	return "something else"
        }

    What does `Bar.AnOp()` return?

    If you're coming from most OOP languages, you're expect AnOp() to dispatch to SomeMethod, which you're expecting to have been inherited
and overridden via the type hierarchy that you thought was in place.

    But there is no type hierarchy, only struct composition. Bar.AnOp() is just a shortform for Bar.Foo.AnOp(), which calls Bar.Foo.SomeMethod(), which
    returns "something." Is that what you expected? Probably not, if you come from an OOP background and are expecting subtype polymorphism, which
    Go doesn't implement.

So let's go back to our original two questions:

1. Does Go meet all the requirements to meet the technical definition of an object oriented programming language?
2. Does Go meet the expectations and assumptions of programmers coming from other object oriented languages?

Go is a language that achieves inheritance without taxonomy, and polymorphism without subclassing.  Because of that,
it meets all the technical requirements needed for OO programming, but that's not how it's designed to be used, and thinking of it
that way is a crutch that might help you get started, but hurt your understanding in the long term. 

If you're asking "is Go object oriented" because you want to write Java in Go syntax, the answer is "no," if you're asking "is Go
object oriented" because it's a question on an exam, the answer is probably "yes," and if you're asking "Is Go object oriented" because
you're in an argument on the internet, the answer is whatever you want it to be.