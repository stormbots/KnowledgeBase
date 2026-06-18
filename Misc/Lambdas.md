---
aliases:
  - Functional Interface
  - Lambda
---
## Synopsis

A "Lambda" is also known as an "anonymous function" . In a general sense, lambda's are just a quirky syntax to create "on the fly" functions, and otherwise work basically like any other function. 

In WPILib code, it's common to want to pass procedures and code blocks to  [[Commands]] and [[Triggers]], allowing those functions to know "how to get new values", rather than just what the values are when you first try to set them up. This is critical for real-time decisions, like "is a game piece loaded?" which you won't know when creating the command. 

WPILib has good further reading that might assist with understanding.
https://docs.wpilib.org/en/stable/docs/software/basic-programming/functions-as-data.html

## Refresher on functions

Functions should be common to most programmers. Let's take a look at the obvious properties of a function to make sure we're well set. 

- A name of the function
- A list of parameters
- And a returned value
- associated code to run

For something like 
```java
public void sayHello(String name){print("hello "+name);}; 
```
we can see that 
- the name is `sayHello`
- There is one parameter, named `name`
- There is code that prints a greeting and the name
- The return type is `void`, or has no return value. 

However, another sneaky property is the function's scope. Scope represents variables and functions that the function has access to that are not strictly part of the function itself.  For Java, this often indicates files in the same class. This too should look familiar:
```java
public class ExampleClass{
	String greeting="sup ";
	public void sayHello(String name){print(greeting+name);}; 
}
```
Here, `greeting` is part of `sayHello`'s scope, but not part of the function itself. This is a property of the function, even if it's not an apparent one. 



## Simple Lambda example

This example takes uses `Commands.run()`, which takes a `Runnable` object, and executes once per robot loop. A `Runnable` simply describes "a function that takes no argument, and returns no values". We'll come back to this. 

We can pass this to a command in two different ways
```java
//Create a named function like we're used to
// and then pass a reference to it
void printSomething(){System.out.println("something");}	
Commands.run(this::printSomething);

//Or, just create it directly.
Commands.run(()->{System.out.println("something");});
}
```

We can see that a lambda looks *very* similar to a function, with effectively the same structure. 
```java
// return type | function name | parameter list | the code block
void             printSomething   ()           {System.out.println("something");}
                                  ()       ->  {System.out.println("something");}
```

The primary difference is the missing function name, and lack of explicit return type. We don't need a name because it's anonymous. We don't need a return type, because it's declared by the Command's argument type: `Runnable`.
The main difference is the `->` ; This is just the special lambda syntax. It might help to consider it as "goes to" or "yields" ; as in the parameter list `(...)` yields a value `{block of code}`.




## Supplier classes
Just as "Runnable" defines "A function that accepts no arguments and returns nothing", there's a whole bunch of standard references for functions that "Take X" and "Return Y". 

These are generally broken down into Suppliers (returns a value) and Consumers (accepts a value). Here's a few.
```java
Runnable // takes nothing, and returns nothing. Just a pure code block.
BooleanSupplier // A function that takes nothing, and returns a boolean
DoubleSupplier // A function that accepts nothing, and returns a double

BooleanConsumer // Function that takes a boolean and returns nothing.
```

That's about it! These helper classes are only necessary as type indicators for function arguments. The functions you're passing in will just be normal functions.


## More complete example

A great reference is the `FunctionalCommand` class. This class takes 4 lambdas, replacing each of the core parts of a [[Commands|Command]] with an appropriate Lambda type. It looks like this. 

```java
FunctionalCommand(
	Runnable, //initialize
	Runnable, //execute
	BooleanSupplier, //isFinished ; returns a boolean
	BooleanConsumer // end; takes a boolean that indicates if it was cancelled.
);
```

Let's say we want to create a command that 
- Print the time it starts
- how long it ran, and if it was successful
- End 5 seconds after it starts

```java
double startTime=0;
new FunctionalCommand(
	()->{//initialize by recording the time
		startTime=Timer.getFPGATimestamp();
		System.out.println(startTime);
	},
	()->{}, //execute: do nothing
	()->{return Timer.getFPGATimestamp()-startTime>=5}, //isFinished
	(cancelled)->{ //our end block
		System.out.println(Timer.getFPGATimestamp()-startTime);
		System.out.println(cancelled);
	}
```

Hooray! This works, and leads us to a useful confirmation: Like any other function, you can access any variable in the current scope in which it was created.

When we combine this feature with [[Subsystems]] and the Factory pattern for commands, we can have variables with a "function scope" serve as data between the components of the command. 

This scope access also is a large value add when creating [[Commands]] inside of [[Subsystems]]. We can just access useful constants, motors, encoders, and helper functions without any extra setup.

## Creating functions that take lambdas

It's often helpful to take functions as arguments. This allows you to isolate the code you're writing from objects that create that data. 

Let's create a function that takes two lambdas, and returns the value of the larger one. You 

```java 
Double myFunction(DoubleSupplier left, DoubleSupplier right){
	var letfvalue = left.getAsDouble();
	var rightvalue = right.getAsDouble();
	if(letfvalue > rightvalue) return left;
	return right;
}
```

This command doesn't do much in practice, but demonstrates the process. A more utilitarian practice is something like a "Drive" command on the robot; Sure, you could take a Joystick object, and then read from that. But, what if you want to feed it values from a Vision process? Vision doesn't have a joystick, so, you need a new entire Drive command for it. If you want to use a rangefinder? Another whole  drive command.

Instead, creating a DoubleSupplier interface means your drive command can get values from arbitrary functions, making them much more generic and helpful.


## Lambdas + Constant Outputs

The other important detail: What if we're trying to run the above `myFunction`, but we just want a constant value on one side? Well, lambdas work here too. We just create a new, lambda that returns a value. Just because it's boring, doesn't mean  it's not helpful!
```java
//Somewhere in our code where we want to do a check
myFunction(()->0.1,joystick::getLeftX)
```

One supplier just evaluates to 0.1 every time it's ran. The other side does something on the human input side. In a realistic sense, this could make some intake idle at a low value to hold a game piece, with a human operator maybe running it with more power to do something useful.

The flexibility of accepting functions vs simply doubles is significant, and will streamline your code development for many advanced features.


