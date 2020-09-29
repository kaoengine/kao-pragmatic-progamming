# Chapter 5. Bend or Break
## 26.-Decoupling and the Law of Demeter
### Minimize Coupling

Be careful about how many other modules you interact with and how you came to interact with them.

Traversing relationships between objects directly can quickly lead to a combinatorial explosion.

```java

	book.pages().last().text().

	// Instead, we're supposed to go with:

	book.textOfLastPage()
```

Symptoms:

1. Large projects where the command to link a unit test is longer than the test program itself
2. "Simple" changes to one module that propagate through unrelated modules in the system
3. Developers who are afraid to change code because they aren't sure what might be affected
4. Meetings where everyon há to attend because no one í sure who will be afedted by a change.
5. Wacky dependencies beteen unrelated modules or libaries.

#### Train Wrecks
![[Pasted image 1.png]]
```java
public void appyDiscount(customer, order_id, discount){
	total = customer.orders.find(order_id).getTotals();
	totals.grandTotal = totals.grandTotal - discount;
	totals.discount = discount;
}
```

```java
public void applyDiscount(customer, order_id, discount) {
 customer
	.findOrder(order_id)
	.findTotal()
	.findAmount()
	.findRule()
	.findAdminApprover()
	.applyDiscount(discount);
 }


public void applyDiscountToOrder(customer){
 orders = customer.orders;
 last = orders.last();
 totals = last.totals();
 amount = totals.amount;
}
```
> This code travel 5 levels of abtractions 

> Tip 45: Tell, Don't Ask 
> You shouldn't make decistions based on the internal state of an object and then update that object. Doing so totally destroy the benefits of encapsulations and, in doing so, spreads the knowledge of the implementations throughout the code. So the first fix for our tain wreack is to delegate the discounting to the total object.
### The Law of Demeter for Functions (LOD)

The Law of Demeter for functions states that  any method of an object should call only methods  belonging to:

```js
class Demeter {
	private A a;
	void m(B b) {
		a.hello(); 							//itself
		b.hello(); 							//any parameters that were passed to the method
		new Z().hello(); 					// any object it created
		Singleton.INSTANCE.hello(); 		// any directly held component
	}
}
```

> Tip 46: Don't chain method Calls  
```js
// This is pretty poor style 
amount = customer.orders.last().totals().amount;

// You should 
orders = customer.orders;
last = orders.last();
totals = last.totals();
amount = totals.amount;
```
##### Exeptions 
There’s a big exception to the one-dot rule: the rule doesn’t apply if the things you’re chaining are really, really unlikely to change. In practice, anything in your application should be considered likely to change. Anything in a third-party library should be considered volatile, particularly if the maintainers of that library are known to change APIs between releases. Libraries that come with the language, however, are probably pretty stable, and so we’d be happy with code such as:
```ruby
people
 .sort_by {|person| person.age }
 .first(10)
 .map {| person | person.name }
```
**Tip 36: Minimize Coupling Between Modules**

### Does It Really Make a Difference?
Using The Law of Demeter will make your code more adaptable and robust, but at a cost:
you will be writing a large number of wrapper methods that simply forward the request on to a delegate. imposing both a runtime cost and a space overhead.
Balance the pros and cons for your particular application.


### Global Data Includes Singletons
> Típ 48: If It’s Important Enough to Be Global, Wrap It in an API

Any mutable external resource is global data. If your application uses a database, datastore, file system, service API, and so on, it risks falling into the globalization trap. Again, the solution is to make sure you always wrap these resources behind code that you control.

### Keep you code be Shy
![[Pasted image 2.png]]
- Don't reveal yourself to other
- Don't interract with too many people 

### Events
An event represents the availability of information. It might come from the outside world: a user clicking a button, or a stock quote update. It might be internal: the result of a calculation is ready, a search finishes. It can even be something as trivial as fetching the next element in a list.
 Whatever the source, if we write applications that respond to events, and adjust what they do based on those events, those applications will work better in the real world. Their users will find them to be more interactive, and the applications themselves will make better use of resources.
 
 #### 1  Finite State Machines
 ![[Pasted image 3.png]]
 #### 2  The Observer Pattern
 In the observer pattern we have a source of events, called the observable and a list of clients, the observers , who are interested in those events.
 An observer registers its interest with the observable, typically by passing a reference to a function to be called. Subsequently, when the event occurs, the observable iterates down its list of observers and calls the function that each passed it. The event is given as a parameter to that call.

 ![[Pasted image 4.png]]
 ![[Pasted image 5.png]]
 There’s not much code involved in creating an observable: you push a function reference onto a list, and then call those functions when the event occurs. This is a good example of when not to use a library.
 The observer/observable pattern has been used for decades, and it has served us well. It is particularly prevalent in user interface systems, where the callbacks are used to inform the application that some interaction has occurred.
 But the observer pattern has a problem: because each of the observers has to register with the observable, it introduces coupling. In addition, because in the typical implementation the callbacks are handled inline by the observable, synchronously, it can introduce performance bottlenecks.

```js
console.clear();

/*************************
 *        Observer       *
 *************************/
 
class Observer {
  constructor(handlers) {
    this.handlers = handlers; // next, error and complete logic
    this.isUnsubscribed = false;
  }
  
  next(value) {
    if (this.handlers.next && !this.isUnsubscribed) {
      this.handlers.next(value);
    }
  }
  
  error(error) {
    if (!this.isUnsubscribed) {
      if (this.handlers.error) {
        this.handlers.error(error);
      }
        
      this.unsubscribe();
    }
  }
  
  complete() {
    if (!this.isUnsubscribed) {
      if (this.handlers.complete) {
        this.handlers.complete();
      }
        
      this.unsubscribe();
    }
  }
  
  unsubscribe() {
    this.isUnsubscribed = true;
    
    if (this._unsubscribe) {
      this._unsubscribe();
    }
  }
}

/*************************
 *       Observable      *
 *************************/

class Observable {
  constructor(subscribe) {
    this._subscribe = subscribe;
  }
  
  subscribe(obs) {
    const observer = new Observer(obs);
    
    observer._unsubscribe = this._subscribe(observer);
    
    return ({
      unsubscribe() {
        observer.unsubscribe();
      }
    });
  }
}

/*************************
 *       fromArray       *
 *************************/

Observable.from = (values) => {
  return new Observable((observer) => {
    values.forEach((value) => observer.next(value));
    
    observer.complete();
      
    return () => {
      console.log('Observable.from: unsubscribed');
    };
  });
}


/*************************
 *        interval       *
 *************************/

Observable.interval = (interval) => {
  return new Observable((observer) => {
    let i = 0;
    const id = setInterval(() => {
      observer.next(i++);
    }, interval);
  
    return () => {
      clearInterval(id);
      console.log('Observable.interval: unsubscribbed');
    };
  });
}

/*************************
 *       fromEvent       *
 *************************/

Observable.fromEvent = (element, eventName) => {
  return new Observable((observer) => {
    const eventHandler = (event) => observer.next(event);
    
    element.addEventListener(eventName, eventHandler, false);
    
    return () => {
      element.removeEventListener(eventName, eventHandler, false);
      console.log('Observable.fromEvent: unsubscribbed');
    };
  });
};

/*************************
 *          map          *
 *************************/

Observable.prototype.map = function (transformation) {
  const stream = this;
  
  return new Observable((observer) => {
    const subscription = stream.subscribe({
      next: (value) => observer.next(transformation(value)),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
    
    return subscription.unsubscribe;
  });
};

/*************************
 *        Examples       *
 *************************/

// ---------------------
// Numbers from array
// ---------------------
const numbers$ = Observable.from([0, 1, 2, 3, 4]);
const numbersSubscription = numbers$.subscribe({
  next(value) { console.log(value); },
  error(err) { console.error(err); },
  complete() { console.info('done'); }
});

setTimeout(numbersSubscription.unsubscribe, 500);

// ---------------------
// Intervals
// ---------------------
const interval$ = Observable.interval(100);
const intervalSubscription = interval$.subscribe({
  next(value) { console.log(value); },
  error(err) { console.error(err); },
  complete() { console.info('done'); }
});

setTimeout(intervalSubscription.unsubscribe, 1000);

// ---------------------
// Click events
// ---------------------
const button = document.querySelector('button');
const clicks$ = Observable.fromEvent(button, 'click');
const clicksSubscription = clicks$.subscribe({
  next(value) { console.log('clicked'); },
  error(err) { console.error(err); },
  complete() { console.info('done'); }
});

setTimeout(clicksSubscription.unsubscribe, 1500);

// ---------------------
// Map
// ---------------------
const mappedInterval$ = Observable
  .interval(100)
  .map((value) => 2 * value);

const mappedIntervalSubscription = interval$
  .subscribe({
    next(value) { console.log(value); },
    error(err) { console.error(err); },
    complete() { console.info('done'); }
  });

setTimeout(mappedIntervalSubscription.unsubscribe, 1500);
```

 #### 3  Publish/Subscribe
 Publish/Subscribe (pubsub) generalizes the observer pattern, at the same time solving the problems of coupling and performance.
 In the pubsub model, we have publishers and subscribers . These are connected via channels. The channels are implemented in a separate body of code: sometimes a library, sometimes a process, and sometimes a distributed infrastructure. All this implementation detail is hidden from your code.
 Every channel has a name. Subscribers register interest in one or more of these named channels, and publishers write events to them. Unlike the observer pattern, the communication between the publisher and subscriber is handled outside your code, and is potentially asynchronous.
 Although you could implement a very basic pubsub system yourself, you probably don’t want to. Most cloud service providers have pubsub offerings, allowing you to connect applications around the world. Every popular language will have at least one pubsub library.
 Pubsub is a good technology for decoupling the handling of asynchronous events. It allows code to be added and replaced, potentially while the application is running, without altering existing code. The downside is that it can be hard to see what is going on in a system that uses pubsub heavily: you can’t look at a publisher and immediately see which subscribers are involved with a particular message.
 Compared to the observer pattern, pubsub is a great example of reducing coupling by abstracting up through a shared interface (the channel). However, it is still basically just a message passing system. Creating systems that respond to combinations of events will need more than this, so let’s look at ways we can add a time dimension to event processing.

 #### 4  Reactive Programming and Streams
 If you’ve ever used a spreadsheet, then you’ll be familiar with reactive programming . If a cell contains a formula which refers to a second cell, then updating that second cell causes the first to update as well. The values react as the values they use change.
 There are many frameworks that can help with this kind of data-level reactivity: in the realm of the browser React and Vue.js are current favorites (but, this being JavaScript, this information will be out-of-date before this book is even printed).


## 26.5- Transforming Programming
All programs transform data, converting an input into an output. And yet when we think about design, we rarely think about creating transformations. Instead we worry about classes and modules, data structures and algorithms, languages and frameworks.
 We think that this focus on code often misses the point: we need to get back to thinking of programs as being something that transforms inputs into outputs. When we do, many of the details we previously worried about just evaporate. The structure becomes clearer, the error handling more consistent, and the coupling drops way down.
 To start our investigation, let’s take the time machine back to the 1970s and ask a Unix programmer to write us a program that lists the five longest files in a directory tree, where longest means “having the largest number of lines.”
 You might expect them to reach for an editor and start typing in C. But they wouldn’t, because they are thinking about this in terms of what we have (a directory tree) and what we want (a list of files). Then they’d go to a terminal and type something like:
 
  $  find   .   -type   f   |   xargs   wc   -l   |   sort   -n   |   tail   -5
  > Tip 49 : Programming Is About Code, But Programs Are About Data

  
 ## OOP vs Functional Programming
 Why Is This So Great?
 Let’s look at the body of the main function again:
 
 ```
 word
 
 |> all_subsets_longer_than_three_characters()
 
 |> as_unique_signatures()
 
 |> find_in_dictionary()
 
 |> group_by_length()
 ```
 It’s simply a chain of the transformations needed to meet our requirement, each taking input from the previous transformation and passing output to the next. That comes about as close to literate code as you can get.
 But there’s something deeper, too. If your background is object-oriented programming, then your reflexes demand that you hide data, encapsulating it inside objects. These objects then chatter back and forth, changing each other’s state. This introduces a lot of coupling, and it is a big reason that OO systems can be hard to change.

> Tip 50 Don’t Hoard State; Pass It Around

In the transformational model, we turn that on its head. Instead of little pools of data spread all over the system, think of data as a mighty river, a flow . Data becomes a peer to functionality: a pipeline is a sequence of code → data → code → data…. The data is no longer tied to a particular group of functions, as it is in a class definition. Instead it is free to represent the unfolding progress of our application as it transforms its inputs into its outputs. This means that we can greatly reduce coupling: a function can be used (and reused) anywhere its parameters match the output of some other function.
 Yes, there is still a degree of coupling, but in our experience it’s more manageable than the OO-style of command and control. And, if you’re using a language with type checking, you’ll get compile-time warnings when you try to connect two incompatible things.

### Inheritance Tax	
You wanted a banana but what you got was a gorilla holding the banana and the entire jungle.
 Joe Armstrong
 Do you program in an object-oriented language? Do you use inheritance?
 If so, stop! It probably isn’t what you want to do.
![[Pasted image 6.png]]
#### Problems Using Inheritance to Share Code
Inheritance is coupling. Not only is the child class coupled to the parent, the parent’s parent, and so on, but the code that uses the child is also coupled to all the ancestors. Here’s an example:
```ruby
class  Vehicle
	def  initialize
 		@speed = 0
  	end 
  	def  stop
	 	@speed = 0
  	end 
  	def  move_at(speed)
 		@speed = speed
  	end 
 end 
 
  class  Car < Vehicle
	  def  info
	  "I'm car driving at  #{ @speed }  " 
	  end 
  end 
 
 
  # top-level code 
 my_ride = Car.new
 my_ride.move_at(30)


```
When the top-level calls my_car.move_at , the method being invoked is in Vehicle , the parent of Car .
 Now the developer in charge of Vehicle changes the API, so move_at becomes set_velocity , and the instance variable @speed becomes @velocity .
 An API change is expected to break clients of Vehicle class. But the top-level is not: as far as it is concerned it is using a Car . What the Car class does in terms of implementation is not the concern of the top-level code, but it still breaks.
 Similarly the name of an instance variable is purely an internal implementation detail, but when Vehicle changes it also (silently) breaks Car .
 So much coupling.
 
 > Tip 51 Don't Pay Inheritance Tax
 
 ### The Alternatives Are Better
 Three suggest techniques that mean you hould never need to use inheritance again:
 - Interface and protocols
 - Delegation
 - Mixins and trais
 
 #### Interface and Protocols
 ```java
 public class Cart implements Drivable, Locatable{
 // Code for class Car. This code must include
 // the functionality of both Drivable and Locatiable
 }
 ```
 > Drivable, Locatable are interfaces, other programming language call them protocols and some call them is **trails**
```java
public   interface  Drivable {
  double  getSpeed();
  void  stop();
 
 }
 
 
 public   interface  Locatable() {
  Coordinate getLocation();
  boolean  locationIsValid();
 
 }
```
*These is no code, what make interfaces and protocol so powerfull is that we can use them as types and any calss that implements the apporiate interface will be compatible with that type*

> **Tip 52: Prefer interfaces to Express Polymorphism**
> Interfaces and protocols give us polymorphims without inheritance.

 #### Delegation 
 ![[Pasted image 7.png]]
 
 #### Mixin and trails
 
## Configuration 
>Let all your things have their places; let each part of your business have its time. Benjamin Franklin , Thirteen Virtues, autobiography
> **Tip 55: Parameterize your app using external Configuration**

Common things you will probably want to put in configuration data include:
 • Credentials for external services (database, third party APIs, and so on)
 •  Logging levels and destinations
 •  Port, IP address, machine, and cluster names the app uses
 •  Environment-specific validation parameters
 •  Externally set parameters, such as tax rates
 •  Site-specific formatting details
 •  License keys
 #### Static configuration 
- Keep it as flatfile(YAML,JSON) or database tables. 
- Whatever form you use, the configuration is read into your application as a data structure, normally when the application starts. Commonly, this data structure is made global, the thinking being that this makes it easier for any part of the code to get to the values it holds.
- We prefer that you don’t do that. Instead, wrap the configuration information behind a (thin) API. This decouples your code from the details of the representation of configuration.
#### Configuration-As-A-Service
 While static configuration is common, we currently favor a different approach. We still want configuration data kept external to the application, but rather than in a flat file or database, we’d like to see it stored behind a service API. This has a number of benefits:
 •  Multiple applications can share configuration information, with authentication and access control limiting what each can see
 •  Configuration changes can be made globally
 •  The configuration data can be maintained via a specialized UI
 •  The configuration data becomes dynamic
 That last point, that configuration should be dynamic, is critical as we move toward highly available applications. The idea that we should have to stop and restart an application to change a single parameter is hopelessly out of touch with modern realities. Using a configuration service, components of the application could register for notifications of updates to parameters they use, and the service could send them messages containing new values if and when they are changed.
 Whatever form it takes, configuration data drives the runtime behavior of an application. When configuration values change, there’s no need to rebuild the code.
#### Don’t Write Dodo-Code
![[Pasted image 8.png]]
Without external configuration, your code is not as adaptable or flexible as it could be. Is this a bad thing? Well, out here in the real world, species that don’t adapt die.
The dodo didn’t adapt to the presence of humans and their livestock on the island of Mauritius, and quickly became extinct. [45] It was the first documented extinction of a species at the hand of man.

## 27.-Metaprogramming
"Out with the details!" Get them out of the code. While we're at it, we can make our code highly configurable and "soft"—that is, easily adaptable to changes.

### Dynamic Configuration
**Tip 37: Configure, Don't Integrate**
### Metadata-Driven Applications
We want to configure and drive the application via metadata as much as possible.
_Program for the general case, and put the specifics somewhere else —outside the compiled code base_

**Tip 38: Put Abstractions in Code Details in Metadata**

Benefits:

* It forces you to decouple your design, which results in a more flexible and adaptable program.
* It forces you to create a more robust, abstract design by deferring details—deferring them all the way out of the program.
* You can customize the application without recompiling it.
* Metadata can be expressed in a manner that's much closer to the problem domain than a general-purpose programming language might be.
* You may even be able to implement several different projects using the same application engine, but with different metadata.

The good news is that thinking in transformations doesn’t require a particular language syntax: it’s more a philosophy of design. You

### When to Configure
A flexible approach is to write programs that can reload their configuration while they're running.

* long-running server process:  provide some way to reread and apply metadata while the program is running.
* small client GUI application: if restarts quickly no problem.

