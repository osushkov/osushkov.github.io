---
layout: post
title: Downsides of Garbage Collection
---

This opinion may be considered backward or anachronistic by many "modern" programmers, but I believe that garbage collection is harmful. To be clear, I don't just mean harmful for performance, but harmful for program design and architecture. It is a sad reality that GC'ed languages currently dominate the landscape. C and C++ are the only mainstream/mature languages that offer manual memory management, with Rust and D providing additional alternatives, if maturity/widespread industry acceptance are not important criteria. I will layout an argument against GC, approaching it from both ends. I will argue that the benefits that GC is thought to provide are overstated, and in fact leads to often overlooked problems. I will also argue that modern “manual” (non-GC) memory management is not nearly as complicated, error prone, and tedious as many people may think. 

![alt]({{ site.baseurl }}/images/gc.jpg)

### Performance
I’ll start off my critique with probably the easiest argument to make: GC performance sucks. Yes, on some benchmarks, some GC based languages (such as Go, or JVM languages like Java and Scala) can perform quite well compared to say C/C++. However, when you dig a little deeper into the issue, you’ll find that this performance comes with a fairly large caveat: you need a lot of spare RAM. If you have 16GB of RAM, and your benchmark has a working set of 1MB, then your Java code will be roughly comparable in run-time to C++. However, as the gap between machine RAM and working set size shrinks, the performance of the GC falls off dramatically. This effect is presented in details by Drew Crawford in his [blog](http://sealedabstract.com/rants/why-mobile-web-apps-are-slow/), and in a paper titled [Quantifying the Performance of Garbage Collection vs. Explicit Memory Management](https://people.cs.umass.edu/~emery/pubs/gcvsmalloc.pdf), by M. Hertz and E. Berger. If your device does not have lots of spare RAM (eg: mobile phones), or if your working set is large (eg: training large Machine Learning models), with GC you’re going to have a bad time. The occasions when GC is fast is when it does nothing at all (when you have plenty of spare memory capacity and it simply doesn’t bother to clean up dead objects). Once it is forced to actually perform cleanup, performance drops precipitously. The unpredictable latency added by the GC process is also a problem in latency sensitive applications such as real time games. I won’t say too much more on the performance problems of GC, as it’s an issue that has been discussed in many other places.


### General Resource Management
Memory is not the only resource that a program has to manage. Garbage collection removes the need for manual memory management, but it does not help with things like managing open file handles, network connections, database connections, mutexes, etc. In fact, the presence of a GC, and hence the absence of predictable object destruction, means that managing non-memory resources becomes harder and more error prone. So with Garbage Collection, although you don’t have to explicitly free allocated memory at the right time, you still have to remember to release file handles, or network connections. As a result, you end up with code blocks that looks like this:

```java
File file = openFile();
// Use file
file.close();
```

This looks a lot like manual resource management to me. Actually, some of you may have realised that the above code is not exception safe. So in reality, your code block would look more like this (new versions of Java introduce a shorthand syntax, but the point remains):

```java
File file = openFile();
try {
    // Use file
} finally {
    file.close();
}
```

A common counter-argument from the pro-GC side is that memory resources account for the majority of system resources that most programs manage. So if GC can address that, then that’s a win right? Not really. Smart pointers and patterns like RAII (Resource Acquisition Is Initialisation) effectively address both memory management and general resource management.


###Smart Pointers and RAII
Many people are grateful for GC because they may remember their early painful experience programming in C or C++, having to remember to call free() or delete on a heap allocated pointer at the right time. Getting it wrong would mean either a memory leak or a free’d pointer dereference, both being less than ideal outcomes. Exceptions added a whole other level of complexity. Several popular tools, such as valgrind, were created with the primary purpose being to detect memory leaks and other memory management problems. In such a situation, automatic garbage collection seems like a small price to pay, right? 

Modern versions of “manual” memory managed languages are very different to what you may remember from those early experiences. The advent and popularisation of various flavours of smart pointers (`unique_ptr` and `shared_ptr` in C++11 for example), have made memory management almost a complete non-issue; without resorting to a garbage collected heap. Idiomatic C++ now basically has completely removed the need for calls to delete. Calls to `delete` are now considered a code smell. Freeing allocated memory happens automatically when a smart pointer exits its scope. What's more, smart pointers and RAII can handle resource management of any type of resource, such as file handles, network connections, or mutexes. Something like a scoped file handle object removes the need for the acquire -> try -> finally -> release pattern we was above. The file handle would simply be released when the encompassing object exists its scope, and would be exception safe by default.


### Lack of Object Ownership Semantics
There’s a good chance that you’ve heard the above arguments before. However, I think my final argument against garbage collection is not a commonly heard one. With manual memory management (using smart pointers), object ownership semantics are easily expressed and become critical. The code makes it clear who is the owner of a particular object, and hence the object’s lifetime. Furthermore, interfaces can explicitly express transferral of object ownership for method arguments and return values. For example, take the below function signatures:

```java
class SomeClass {
    void fooFunc1(unique_ptr<Object> obj);
    void fooFunc2(Object* obj);

    unique_ptr<Object> barFunc1(void);
    Object* barFunc2(void);
};
```

On the surface there is very little difference between the corresponding functions. For foo, both take essentially a pointer to an object, and Bar returns a pointer to an object. However, in idiomatic modern C++, fooFunc1 indicates that it will take ownership of the passed object, whereas the second function does not take ownership and will not keep the passed in pointer around for longer than the lifetime of the function. A similar semantic is expressed by the bar functions. barFunc1 returns an owning pointer to an object, indicating that ownership of the encapsulated object is tranferred to the caller, and that SomeClass will not retain a reference to it. barFunc1 returns a naked pointer, indicating no transfer of ownership. 
These ownership semantics can be important information when designing and using a particular system/interface, but is impossible to effectively and succinctly express in a garbage collected language such as Java. 

Now, an argument can easily be made that ‘who cares about object ownership semantics, it’s a low level detail that should be abstracted away’. It is true that one can simply ignore this aspect of programming, let the GC figure out ownership and object lifetime (or go the Object-C route and treat all object references as reference counted shared pointers internally). However, I believe that the process of forcing the programmer to explicitly consider object ownership when designing interfaces and classes promotes more robust and well thought out designs. When you find yourself unsure of who actually owns an object, maybe being tempted to simply use a shared_ptr to get around the obstacle, this should set off alarm bells and prompt you to rethink your design. On numerous occasions I’ve found myself refactoring a design into a much cleaner one after being prompted to do so by an awkward transfer of ownership, or by finding myself in a situation where there was no clear single owner of an object. These type of “code smells” are largely absent when programming in a GC’ed language such as Java, and as a result I believe overall class and interface design suffers.


### Conclusion
So having made a number of arguments against GC and in favour of assisted manual memory management doesn't mean I think there is no place for garbage collection. For many applications it is the natural choice. However, the current popular attitude seems to be that languages with manual memory management are only appropriate for low level software (micro-controllers, OS kernels), or for high performance/low latency applications (games). I think that “manually” memory managed languages have a place in high-level application programming, as they allow the explicit expression of object ownership semantics. This is a useful feature for creating well designed software programs.

