JVM Memory Management - 
How objects allocated.
How Objects garbage collected.
How does Garbage Collector determine Live objects.

MIND MAP - 
1) Why GC ?
2) Different Types of GCs - Do nothing , Reference Counting, Mark and sweep, Coying, Generational, Incremental.
        GC in oracle JVM 
        JVM are combination of these GCs - Generational GC, Copying, Mark and sweep 
3) Things to consider - Stop the world events, throughput, Memory fragmentation.
   Memory - Eden Space , survivor space, tenured space , permanent space.
4) Young Generation - Eden Space , survivor space.
5) Minor Garbage collection (how it happens using youg Generation), Major Garbage Collection (What is it).
6) ASIDES before going to Major Garbage Collection- 
             - How to objects come in old generation
             - How objects are allocated by JVM. - bump the pointer, TLBs
             - How objects are choosen to be live (by young GC and old generation GC).
             - How choosing of live objects is made easy for Young Gen GC. (What Is a Cardtable and How Is It Used in Garbage Collection...write barries.)
             - a prediction about Old Gen mechanism of watching live objects.


# JVM Memory Management

## :bulb: Introduction 

We learn how to interact with JVM externally i.e using tools given by JVM or internally using some classes provided by JVM.  

### :bulb: Garbage Colletion .. Why GC?
Imagine,  
> Account acc = new Account();  
The question will arise who will delete this object? Creator or user?  

**1) CREATE AND FORGET:**  
No need to remember to delete. GC will do it. We don't need to free the memory as in C++ and C.  

**2) USE AND FORGET:**  
No need to ask "Should I delete ?"  

> Account acc = getAccount();  
Thus either the receiver or the giver of acc object can delete i.e the class that implements getAccount()  

If neither deletes it -> memory leak  
In C++ the method who is getting this object need to free the memory, so the user of object need to have in mind when to delete object but in java the user does not need to think about object freeing while using an object
If both delete it -> null pointer exception somewhere else which will be very hard to track.  

**3) USE WITH CONFIDENCE:**   
Objects will not vanish or corrupt behind your back. In C++ some other thread may delete the object which will create null ptr exception later.

### GC PROMISE  
- Claim no live objects  
- No promise about dead objects.  
So dead objects may be present in memory. BUT no live object will be deleted or reclaimed. 
We don't know when GC will run, whether it will run before our application ends or not.


### :bulb: Different Types of Garbage Collection
There are different types of GARBAGE COLLECTORS -  
**1) Do nothing** - Not freeing any memory. Just guarenting that no live objects will be reclaimed.

**2) Reference Counting** - for a variable two functions - 
    - addReference - increment reference count. 
    - release - decrements reference count. 
When reference count reaches zero, object can clean itself up. In this we’ll not be able to delete cyclic references.

**3) Mark and sweep** - Has 2 phases ....Mark phase walks through all live memory marking it alive and sweep phase removes all unused memory. This leads us to memory which can be fragmented. Compaction is done by rearranging the marked memory.

**4) Copying** - Work hand in hand with mark and sweep garbage collection. Marked objects are copied to a buffer (new space) before sweep phase runs on the old space, hence the memory is no longer fragmented.

**5) Generational** - If an object survives one GC, then GC would not look at it again for a while. This improves performance of GC.

**6) Incremental** - Does not look at all the memory all the time during Garbage collect. so kinda like Generational.

So GC are of many types. So we tend to have a mixture of all these.  

Now let's have a look at some of these in detail.  

### 2. Reference Counted Garbage Collection
In reference counting, we have a problem of circular references where an object1 references object 2 and object2 also references object1 and they have no external reference. This is also known as Island of Isolation.  
These cyclic references won't be removed, although there is no reference that can be used to access these objects.

### 3. Mark and Sweep GCs

Typically 3 phases -  
**MARK** - Identifying objects still in use.
Start from the root set and following other references from nodes of memory, GC mark the live memory.
In case of cycle in memory and not reference from root set .. No problem because we’ll not be able to reach that cycle as no external references. 

What kinda nodes are rootes .. temme ? Yeah Think bachaaaa. (Gyanam na so easy Asti)

Main class .. main method scope, Claases objects, JVM (which tell which objects are always on ).

![noImage](./img/MarkAndSweep1.png) 

![noImage](./img/MarkAndSweep2.png)

**SWEEP** - Removes unused objects.

![noImage](./img/MarkAndSweep3.png)

**COMPACT** - To compact the memory. So physical addresses of memory changed. a references rearranged accordingly in root set

In java we don't have physical addresses of memory. Objects internally manages them.

![noImage](./img/MarkAndSweep4.png)


### 4. Copying GCs 

Things are a little different.
When memory for a buffer gets full. GC run and mark live memory.

![noImage](./img/CopyingGC1.png)

Then it copies it to other buffer and rearranges references in root set.

![noImage](./img/CopyingGC2.png)

Then removes dead memory from previous memory.

![noImage](./img/CopyingGC3.png)

After the copy , we end up with a compacted copy of the data in new space data and a (hopefully) large, contiguous area of memory in new space in which we can quickly and easily allocate new objects.
The next time we do garbage collection (i.e when the space in which we are allocating memory from now gets full), the roles of old space (from space) and new space (to space) will be reversed.


### 5. Generational GCs

GENERATIONAL COLLECTORS
There are different generations - Younger , older.  
Idea is once an object survives one GC. it is promoted to a different generation.  
And GC will sweep through the younger generation more often than the older generation.  


Let's see how thihs happens:  
So in young generation we have allocated memory. We have old generation, where there maybe alive maynot be alive.
Once GC runs, the survivors of that are moved to old generation.

![noImage](./img/GenerationalGC1.png)

![noImage](./img/GenerationalGC2.png)


And diff environment have diff number of generations. 
Java has two: Young and old generations.
.Net has three.


## :bulb: How Garbage Collection Works in the Oracle JVM

### Introducing the Players

**Things to consider**  
- Stop the world events - its when GC pauses the entire application and at that point it collects garbage. We want to try and minimise these events. These events needed as when there is less memory left, application should not allocate more memory to objects.
- Memory fragmentation - Does GC Defragments memory all at one or leave it for later stage. or it leaves memory fragmented on the basis that it may lead to lower cost than defragment it.  
- Throughput - how quickly can GC run, How quickly can it collect garbage and how it effects behaviour of application.  

**Gcs in JVM are combination of these GCs**  
- Generational GC
- Copying
- Mark and sweep 

**Multi-core** 
We also need to think about is if you are running on a multi core machine, So can GC run in parallel with application . Java provides these types of GC as well.


### The Basics of Garbage Collection in the Java Virtual Machine

![noImage](./img/JVM-MemoryStructure.png)

JVM has a 'Young Generation' and 'Old Generation'.  

**Young Generation:**  
Eden space - Most initial objects are allocated in this space.
Survivor Spaces - Young generation has two 'survivor' spaces
Objects that survive a GC are copied to a new survivor space and objects in original survivor space also gets copied into this new survival space. So all the objects surviving GC live in one of these survivor spaces i.e. only one survivor space is in use at a time.

**Old Generation:**    
When an object survives number of GCs. Then runtime decides that that object will essentially live forever and hence moves it to the old generation.  

**_Most objects live for a very short time. The 'turtle' theory of Garbage collection i.e. you die young or live 'forever'._**  

**Permanent Space**   
Here live things used by Java runtime. Things like class information is stored here. This is never GCied.



### :bulb: MINOR GARBAGE COLLECTS AND MAJOR GARBAGE COLLECTS

**Minor Garbage collection** - When GC collects objects in Young generation. How this works?  

- Objects that were allocated into Eden space
   When GC runs, objects from Eden space are copied to 'newer' survivor space   
   Objects from 'older' survivor space also copied to 'newer' survivor space  
   Survivor spaces are swapped  
- Any new object created is allocated to Eden space.
   
![noImage](./img/MinorGarbageCollection2.png)

![noImage](./img/MinorGarbageCollection3.png)

![noImage](./img/MinorGarbageCollection4.png)

![noImage](./img/MinorGarbageCollection5.png)

![noImage](./img/MinorGarbageCollection6.png)

![noImage](./img/MinorGarbageCollection7.png)

**:bulb: Major garbage collection**  
- Triggered when the old/tenured generation is full. 
- Collects old and young generations (this is really a 'full GC'... JVM must be going from rootSet to all nodes .. marking them live). In such a case the young generation is collected first followed by the old generation. If the old generation is too full to accept the content of the young generation, the young generation GC is omitted and the old generation GC is used to collect the full heap, either in parallel or serial. Either way the whole heap is collected with a stop-the-world event.
- It is slow as Major GC has to go through large sections of heap. It's also possible that, The memory allocated had been paged. So it has again to be paged back in.
- Its also possible to allocate objects directly into the old generation. No direct way of doing it. But we can set option on the JVM called PretenureSizeThreshold.  (i.e if size of object is above threshold  .. put it in tenure space i.e old generation)


### THESE ARE COLLECTION THAT HAPPEN.. its not necessary that one GARBAGE COLLECTOR does both young and Old collection. There may be ONE COLLECTOR  or COMBINATION OF DIFFERENT TYPES OF COLLECTORS which does Major and Minor Collection.

### :bulb: We are going to be learning types of GCs used in JVM. But before that lets learn a bit of things about Java objects  allocation , transfer, marking live etc :
 - How to objects come in old generation
 - How objects are allocated by JVM.
 - How objects are choosen to be live (by young GC and old generation GC).
 - How choosing of live objects is made easy for Young Gen GC. 

After this we will study different types of GCs (which does Old generations collection OR both young and old generation colection). OKAY ....

So ....Major GC collects old gen. So question now comes.. What are moments when old gen gets objects.

### 1) When will JVM promote the objects to old generation :question:  
- After a certain number of garbage collects.
- If survivor space is full.
- If JVM has been told to always create objects in old space
    **-XX:+AlwaysTenure** flag to JVM  
    
![noImage](./img/MajorGarbageCollection3.png)

![noImage](./img/MajorGarbageCollection4.png)

![noImage](./img/MajorGarbageCollection5.png)

Now before even all this.. When an object is made it needs to be allocated space.. right? .. Allocation to Young generation should be really fast as object is getting space in it when it gets created.. And objects that are already mostly gets copied to older generation . So Lets dive into this.

And also we need to know How do we choose whether an object is live or not. 

### 2) How Allocations Work in the Java Virtual Machine
How does the allocation of Memory happens in java (any of one can happen in either old or young generation)
1) allocate and incr pointer.
2) But in multithreaded environment two threads may compete for the same piece of memory. So we'll be needing locks and locks are expensive. So TLABS (Thread-Local Allocation Buffers) used. This allows each thread to have a small portion of its Eden space that corresponds to its own share. As each thread can only access to their own TLAB, even the bump-the-pointer technique will allow memory allocations without a lock. 

![noImage](./img/MemoryAllocation1.png)

![noImage](./img/MemoryAllocation2.png)

![noImage](./img/MemoryAllocation3.png)

![noImage](./img/MemoryAllocation4.png)

![noImage](./img/MemoryAllocation5.png)

![noImage](./img/MemoryAllocation6.png)


### 3) How do we choose an object to be live and others not. Does Young GC has to go every node to get live Young objects ?? And how does old GC see live objects?

-------- ASIDE START-------- 

 JVM STACK AREA 
For every thread, JVM creates a separate stack at the time of thread creation. The memory for a Java Virtual Machine stack does not need to be contiguous. The Java virtual machine only performs two operations directly on Java Stacks: it pushes and pops frames. And stack for a particular thread may be termed as Run – Time Stack. Each and every method call performed by that thread is stored in the corresponding run-time stack including parameters, local variables, intermediate computations, and other data. After completing a method, corresponding entry from the stack is removed. After completing all method calls the stack becomes empty and that empty stack is destroyed by the JVM just before terminating the thread. The data stored in the stack is available for the corresponding thread and not available to the remaining threads. Hence we can say local data is thread safe. Each entry in the stack is called Stack Frame or Activation Record.  

The Java stack is composed of stack frames (or frames). A stack frame contains the state of one Java method invocation. When a thread invokes a method, the Java virtual machine pushes a new frame onto that thread's Java stack. When the method completes, the virtual machine pops and discards the frame for that method.

-------- ASIDE END-------- 

**Live roots -**
1) if reference coming from stack frame they must be live.  
2) Static variables - not part of an instance . kinda global. So should be kept live.  
3) if using Java Native interface (JNI) or synch monitors doing locking , Those are also live  

**What objects are kept live during GC ?**  
- So any object with reference form live root is kept live during GC.
- References from live rooted objects are also kept live. (Even if references from old gen to young ones, they are also kept live. Why? Since it is present in the old gen .. it is considered live thus its childeren should also be considered live, therefore all the objects in the young generation that are referred from this old gen object must be preserved.)

But how. Does it start from root Set and while traversing also scans old gen space ..?? 
No.. Uses Card tables.

### 4) How choosing of live objects is made easy for Young Gen GC....... What Is a Cardtable and How Is It Used in Garbage Collection

![noImage](./img/LiveobjectsManagement1.png)

![noImage](./img/LiveobjectsManagement2.png)

![noImage](./img/LiveobjectsManagement3.png)

My understanding is that an entry in the card table is set when an object in young generation is allocated that is referenced by something in the old generation
When a write to a reference (to an object in young generation) happens, These writes go through something called write barrier.
And this write barries triggers code in JVM, this code updates an entry in a table called card table
So minor GC sees card table and load the memory corresponding to references present in card table and follow references in that memory , to mark them in use.

![noImage](./img/LiveobjectsManagement4.png)

![noImage](./img/LiveobjectsManagement5.png)

![noImage](./img/LiveobjectsManagement6.png)

![noImage](./img/LiveobjectsManagement7.png)

![noImage](./img/LiveobjectsManagement8.png)

![noImage](./img/LiveobjectsManagement9.png)


### :bulb: Yess the time has come.. Lets study different Garbage Collection used by JVM . All of these do Old Generation collection atleast. These may be used alone or in combination with some other type of GC to do both Young And Old Collections.
They are of 5 types.
1) Serial GC
2) Parallel GC
3) Parallel Old GC (Parallel Compacting GC)
4) Concurrent Mark & Sweep GC  (or "CMS")
5) Garbage First (G1) GC

###  Serial Versus Parallel Garbage Collectors

(https://www.cubrid.org/blog/understanding-java-garbage-collection)  

Now we look at different types of GCs
Serial , Parallel and parallel old collector work in same way. Uses mark-copy for the Young Generation and mark-sweep-compact for the Old Generation. Eden , survivor . old space and mark-sweep-copying(or compact) (for defragmentation) algo.
Where they differ is the amount of concurrency each collector has.
### MGC1) Serial collector
Single threaded - it means it is a stop the world collector i.e we stop everything and run GC.  

![noImage](./img/SerialCollector.png)

### MGC2) Parallel collector - Multiple threaded for minor collection and single for major.  So less down time i.e less stop the world time.  

![noImage](./img/parallelCollector.png)

### MGC3) Parallel old collector - multiple for both.  

![noImage](./img/ParallelOldCollector.png)

### MGC4) concurrent mark and sweep (https://plumbr.io/handbook/garbage-collection-algorithms-implementations/concurrent-mark-and-sweep)
- Only collects old space. Only collect during major GC.
- no bump the pointer allocation.... fragments the heap and manages sets of free lists for each of the fragments. It tries to allocate the object in one of these fragments and so it has to check these free lists and update these free lists. So it causes the collector to run more slowly.
- However these are designed to be low latency collectors. So its throughput is much higher with this than parallel and parallel old.


![noImage](./img/ConcurrentMarkAndSweep1.png)

![noImage](./img/ConcurrentMarkAndSweep%20Before%20GC.png)

![noImage](./img/ConcurrentMarkAndSweep%20After%20GC.png)

![noImage](./img/ConcurrentMarkAndSweep%20After%20GC%20CMS.png)

![noImage](./img/ConcurrentMarkAndSweep2.png)

It goes through different phases -
Why does it has to stop the world (Ofcourse there is no space in Old generation. Where will you allocate a new object... i.e Why stop the world so that no new object is created and we dont have to put objects in old generation.)

Just to bear in mind – in real world situation Minor Garbage Collections of the Young Generation can occur anytime during concurrent collecting the Old Generation. In such case the major collection records seen below will be interleaved with the Minor GC events.
1) Initial mark - Mark objects taken from root refs. It does not traverse the graph . it only takes direct child of roots, So that we have minimal stop the world time. The goal of this phase is to mark all the objects in the Old Generation that are either direct GC roots or are referenced from some live object in the Young Generation. 

Now we need to traverse these nodes. But we can do that concurrently with application (i.e Java threads).

2) Concurrent mark - Traverse object graph looking for live objects from objects from previous phase. ANY OBJECT (from young gen to old gen) ALLOCATION MADE DURING THIS PHASE IS AUTOMATICALLY MARKED LIVE.
We wanna go towards - marking all live objects in the Old Generation.
During One traversal of graph it may be so that references of some nodes are changed as this is happening concurrently with Java threads i.e new references to some objects are made or references to some object are removed. JVM mark these objects/nodes as "DIRTY" and then it again traverses graph for these changed nodes marking nodes as live or unlive.

This happens iteratively until one of the abortion conditions (such as the number of iterations, amount of useful work done, elapsed wall clock time, etc) is met. This phase may significantly impact the duration of the upcoming stop-the-world pause.

3) Remark - In the remark step, the objects that were newly added or stopped being referenced (because java threads were running concurrently) in the concurrent mark step are checked and then corresponding entries are marked live or not live. It has to be stop the world to guarantee no more objects created while this step is  executing. This happens quickly because of re-iterative characteristic of previous step which leaves very less work for this step.

The goal of this stop-the-world phase is to finalize marking all live objects in the Old Generation. Since the previous preclean phases were concurrent, they may have been unable to keep up with the application’s mutating speeds. A stop-the-world pause is required to finish the ordeal.

Usually CMS tries to run final remark phase when Young Generation is as empty as possible in order to eliminate the possibility of several stop-the-world phases happening back-to-back.

4) concurrent sweep - collect object not marked live.i.e after previous phase it knows objects that were not marked live i.e these nodes must behaving flag of “NOT MARKED LiVE" . So does not matter even if new objects are created during this process. It will only collect these unmarked  objects. 

5) Resetting - resets everything for next round.


### MGC 5) The G1 Garbage Collector
G1 Garbage collector - planned replacement of CMS
instead of having Eden and tenure OR
Young and old memory. We now break all memory space into regions

https://www.dynatrace.com/news/blog/understanding-g1-garbage-collector-java-9/

Granted, if one wants to collect the entire heap, the G1 has to do the same amount of work as any other GC, but this is where the G1 shines because it doesn’t have to collect the entire heap. It doesn’t even have to collect an entire generation. It can select any number or combination of regions to collect. To optimize collection time, it always selects regions that are full (or almost full) of garbage and thereby minimizes the amount of work it has to do to free heap space for subsequent allocations. Other GCs always collect an entire generation, meaning their run-time complexity often depends on the total heap size. In the G1 case however, this depends on the amount of live objects because memory can be freed without handling an entire generation. Ideally, when the heap is big enough, some regions will always be completely full of garbage, making it easy to collect them.

In the Java world, we already know about concurrent collections from the Concurrent Mark & Sweep GC (CMS). However, the CMS can only collect the old generation concurrently, it still needs to halt the application to collect the young generation.

The G1 only stops the application at the beginning of the GC to do some quick bookkeeping before it immediately resumes the application. This phase is called the “Initial Mark”. Then, while the application is executing, the GC will follow all references and mark life objects (“Concurrent Mark” phase). When this is done, the application is suspended again, and a final cleanup is made (“Final Mark” phase) before selecting a few regions and collecting them (“Evacuation” phase). As the evacuation phase is fast, especially for large heaps, the G1 usually outperforms other GCs in terms of suspension time of the executed application.


![noImage](./img/G1Collector1.png)

![noImage](./img/G1Collector2.png)

![noImage](./img/G1Collector3.png)

![noImage](./img/G1Collector4.png)

![noImage](./img/G1Collector5%20Young%20GC1.png)

![noImage](./img/G1Collector5%20Young%20GC2.png)

![noImage](./img/G1Collector6%20Old%20GC1.png)

![noImage](./img/G1Collector6%20Old%20GC2.png)


### :bulb: Which Collector to use?

![noImage](./img/Differrent%20GCs1.png)

![noImage](./img/WhichCollector1.png)

![noImage](./img/WhichCollector2.png)


## Java Reference Classes

### Introduction to Java Reference Types

Special references available
special because JVM handles them differently and is aware of their existence.
if coming to out of memory exception then softly referenced objects will be GCied
Soft reference is able to keep object alive under certain circumstances but weak reference will never keep an object alive. So whenever GC runs, then an object with no strong and Soft reference will be GCied.

![noImage](./img/Java%20references%20intro1.png)

![noImage](./img/Java%20references%20intro2.png)


### How Java References Work


![noImage](./img/Using%20Reference%20types1.png)

![noImage](./img/Using%20Reference%20types2.png)

![noImage](./img/Using%20Reference%20types3.png)

![noImage](./img/Using%20Reference%20types4.png)

![noImage](./img/Using%20Reference%20types5.png)

![noImage](./img/Using%20Reference%20types6.png)


How these ref work
Person person = new Person ();
WeakReference<Person> wr = new WeakReference<Person>(person);
wr is reference to an object of type WeakReference <Person> and this object internally has a weak reference to Person object (shown as dotted line in pic) which has a strong reference person pointing to it.
Now if i do Person p = wr.get()
I will get a strong reference to Person object

now if i do person = null then we Person object now has no strong reference. But we still have a weak reference. and now if we do wr.get() it will give strong reference to it. So as far as there is  no GC till that time  weakReferenceObject can be used to get a strong reference to Person object.
However if we did GC just after making person = null;
person=null;
System.gc();
at this point weakReference object is not pointing to anything.
so if we do wr.get() , it returns null


### How Different Reference Types Used in Your Code

Usages of reference types

**Weak reference** - used to associate metadata with our types.. need not be real metadata data but some extra data that we need to the type.
ex - if we have a type marked as final we can't extend it. But in certain circumstances we would like to extend it and add extra fields to it. and weak hash map and weak Reference provide us the way to do that.  

Weak Hash Map-  
to have meta data for types which are final.  
hash map in which key is a weak reference to the object (which is of type final) and value is reference to metaData.  
So if object has no strong references, weak reference to that object which is key now is released and the meta data corresponding to that is also released  

What if we have created HashMap with key as normal object to put meta data ? What disaster have we done????   As we have put key as a strong reference we would not be able to GC the object referenced by key. This is the problem.


![noImage](./img/WeakHashMap1.png)

![noImage](./img/WeakHashMap2.png)

![noImage](./img/WeakHashMap3.png)

![noImage](./img/WeakHashMap4.png)

![noImage](./img/WeakHashMap5.png)

![noImage](./img/WeakHashMap6.png)

![noImage](./img/WeakHashMap7.png)

![noImage](./img/WeakHashMap8.png)


**Soft reference** - used for caching
Idea is that when we have a large object (maybe an image object that we require to get from a server) , I create a strong reference to it AND a soft reference to it.
When object not needed I can make strong reference to null.
So i can retrieve this object if i need to by going through soft reference. In case of memory pressure this will go away and when i call get on soft reference i will return null.

Soft reference caching - ( SEE PICS ) not that good as GC manages everything . so we have no control over cache. We cannot used LRU or MRU list. So control of when an element will leave the cache is not our hands.
So soft references can act only as simple cache.. not good for caching in general.


**Phantom reference** -
least used reference. allows us to interact with GC. We can use this to monitor when an object is being collected and do some extra work when that object is being collected. This allows us to interrupt the GC. 

finalize in java - wherever u can use finalize u can use phantom reference . finalizers are expensive

When we call get method on phantom refs they always return null.
Used instead of finalizers which can be expensive as when an object is constructed the runtime has to know about it as it need to call finalize method on it once the object is no longer reachable. So it means there is a code that runs that tells The GC that we have an object that is finalizable. There is a reference to that finalizable object kept by the runtime. So these objects will survive at least one GC as the runtime itself goes to reference these things and runtime must at some point call finalize method before finally releasing the object.

We have more control over when the final clean up of the object happens. It's not under garbage collector, it more under our control.

![noImage](./img/phantomReference1.png)

![noImage](./img/phatomReference2.png)

![noImage](./img/phatomReference3.png)

![noImage](./img/phatomReference4.png)

![noImage](./img/phatomReference5.png)

![noImage](./img/phatomReference6.png)

![noImage](./img/phatomReference7.png)


**Using ReferenceQueue**  
When creating an object we can pass reference queue as a parameter. But when creating phantom refs queue is necessary to pass as param.
Let's say we make a weak reference for an object. when an object has no strong refs attached to it, weak refs are enqueued to queue.
useful to attach cleanup mechanism
RefQueue has poll and remove method
Poll - we are polling the queue. if has any object return it or return null.
Remove - it blocks, untill therre is reference on the queue. we can block for some few second using timeout in remove too. it returns null if no ref type till the timeout.

can attach clean up code. When we dequeue an object we call its cleanup method. at this time we know object has been garbage collected and we know that we can cleanup any data we want.
When system is GCied , weak reference is added to ref queue if no strong refs is attached to object.
We can do the cleanup mechanim on a background thread using executorService.


Rehesyamain link https://dzone.com/articles/weak-soft-and-phantom-references-in-java-and-why-they-matter

![noImage](./img/Referencequeue1.png)

![noImage](./img/Referencequeue2.png)

![noImage](./img/Referencequeue3.png)

![noImage](./img/Referencequeue4.png)

![noImage](./img/Referencequeue5.png)

![noImage](./img/Referencequeue6.png)

![noImage](./img/Referencequeue7.png)



