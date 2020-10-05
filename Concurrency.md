# Concurrency 
### Define:
 - **Concurrency** is when the execution of two or more pieces of code act as if they run at the same time. **Parallelism** is when they do run at the same time.
- To have concurrency, you need to run code in an environment that can switch execution between different parts of your code when it is running. This is often implemented using things such as fibers, threads, and processes.
- To have parallelism, you need hardware that can do two things at once. This might be multiple cores in a CPU, multiple CPUs in a computer, or multiple computers connected together.
![[Pasted image 10.png]]

## Everything is concurrent 
In the real world, things are asynchronous: user are interracting, data fetched, external services are biging called, all at the same time. If you force this process to be serial, with one thing happening, then the next, and so on, your system feels sluggish and you’re probably not taking full advantage of the power of the hardware on which it runs.

### Challenges
![[Pasted image 20200929102152.png]]
How many tasks do you perform in parallel when you get ready for work in the morning? Could you express this in a UML activity diagram? Can you find some way to get ready more quickly by increasing concurrency?
![[Pasted image 20200929102258.png]]

The ideal things to split this way are pieces of work that are relatively independent—where each can proceed without waiting for anything from the others. A common pattern is to take a large piece of work, split it into independent chunks, process each in parallel, then combine the results.

### Share state is incorrect state
![[Pasted image 20200929110403.png]]

### Doctor, It Hurts…
They may call it mutexes (for mut ual ex clusion), monitors, or semaphores. These are all implemented as libraries.

However, some languages have concurrency support built into the language itself. Rust, for example, enforces the concept of data ownership; only one variable or parameter can hold a reference to any particular piece of mutable data at a time.
 
You could also argue that functional languages, with their tendency to make all data immutable, make concurrency simpler. However, they still face the same challenges, because at some point they are forced to step into the real, mutable world.
 
If you take nothing else away from this section, take this: concurrency in a shared resource environment is difficult, and managing it yourself is fraught with challenges.

Which is why we’re recommending the punchline to the old joke:
Doctor, it hurts when I do this. Then don’t do that.
The next couple of sections suggest alternative ways of getting the benefits of concurrency without the pain.

### Actor and Processes 

## 28.- Temporal Coupling
Two aspects of time:

* Concurrency: things happening at the same time
* Ordering: the relative positions of things in time

We need to allow for concurrency and to think about decoupling any time or order dependencies.
Reduce any time-based dependencies

### Workflow
> Tip 56: Analyze Workflow to Improve Concurrency

Use [activity diagrams](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Activity_conducting.svg/2000px-Activity_conducting.svg.png) to maximize parallelism by identifying activities that could be performed in parallel, but aren't.


**Tip 39: Analyze Workflow to Improve Concurrency**

### Oppertunities for Concurrency
Activity diagrams show the potential areas of concurrency, but have nothing to say about whether these areas are worth exploiting. For example, in the piña colada example, a bartender would need five hands to be able to run all the potential initial tasks at once.
![[Pasted image 11.png]]

### Architecture
*Remember the distinction: concurrency is a software mechanism, and parallelism is a hardware concern.*

Balance load among multiple consumer processes: **the hungry consumer model.**

In a hungry consumer model, you replace the central scheduler with a number of independent consumer tasks and a centralized work queue. Each consumer task grabs a piece from the work queue and goes on about the business of processing it. As each task finishes its work, it goes back to the queue for some more. This way, if any particular task gets bogged down, the others can pick up the slack, and each individual component can proceed at its own pace. Each component is temporally decoupled from the others.

**Tip 40: Design Using Services**

### Nonatomic Dessign
![[Pasted image 12.png]]
The problem here is not that two processes can write to the same memory. The problem is that neither process can guarantee that its view of that memory is consistent. Effectively, when a waiter executes display_case.pie_count() , they copy the value from the display case into their own memory. If the value in the display case changes, their memory (which they are using to make decisions) is now out of date.
This is all because the fetching and then updating the pie count is not an atomic operation: the underlying value can change in the middle.

### Design for Concurrency
Programming with threads imposes some design constraints—and that's a good thing.

* Global or static variables must be protected from concurrent access
* Check if you need a global variable in the first place.
* Consistent state information, regardless of the order of calls
* Objects must always be in a valid state when called, and they can be called at the most awkward times. Use class invariants, discussed in Design by Contract.

### Cleaner Interfaces
Thinking about concurrency and time-ordered dependencies can lead you to design cleaner interfaces as well.

**Tip 41: Always Design for Concurrency**

### Deployment
You can be flexible as to how the application is deployed: standalone, client-server, or n-tier.

If we design to allow for concurrency, we can more easily meet scalability or performance requirements when the time comes—and if the time never comes, we still have the benefit of a cleaner design.

## 29.-It's Just a View
### Publish/Subscribe
Objects should be able to register to receive only the events they need, and should never be sent events they don't need.

Use this publish/subscribe mechanism to implement a very important design concept: the separation of a model from views of the model.

### Model-View-Controller
Separates the model from both the GUI that represents it and the controls that manage the view.

Advantage:

* Support multiple views of the same data model.
* Use common viewers on many different data models.
* Support multiple controllers to provide nontraditional input mechanisms.

**Tip 42: Separate Views from Models**
### Beyond GUIs
The controller is more of a coordination mechanism, and doesn't have to be related to any sort of input device.

* **Model** The abstract data model representing the target object. The model has no direct knowledge of any views or controllers.
* **View** A way to interpret the model. It subscribes to changes in the model and logical events from the controller.
* **Controller** A way to control the view and provide the model with new data. It publishes events to both the model and the view.


## 30.-Blackboards
A blackboard system lets us decouple our objects from each other completely, providing a forum where knowledge consumers and producers can exchange data anonymously and asynchronously.

### Some key features of the blackboard approach are
- None of us need ot know of the exsitence of any other mind. We watch the board for new information and add thier findings.
- We be trainned in different disciplines, may have different levels of eduaction and expertise, and may not evn work in the same precint. We sahre desire to solve the prolem, but that's all.
### Blackboard Implementations
With Blackboard systems, you can store active objects—not just data—on the blackboard, and retrieve them by partial matching of fields (via templates and wildcards) or by subtypes.

Functions that a Blackboard system should have:

* **read** Search for and retrieve data from the space.
* **write** Put an item into the space.
* **take** Similar to read, but removes the item from the space as well.
* **notify** Set up a notification to occur whenever an object is written that matches the template.

Organizing Your Blackboard by partitioning it when working on large cases.

**Tip 43: Use Blackboards to Coordinate Workflow**

