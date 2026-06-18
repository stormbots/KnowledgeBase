### Goals:
- Provide human input to the robot
- Have functional motor control


# Coding up a controller

Most example projects have a controller. In some examples it's in Robot.java, but in most it's in RobotContainer.java . We'll generally use the latter.

There's a few different ways to construct controller objects, but the common one is CommandXBoxController

```java
public class RobotContainer{
	private final CommandXboxController joystick = new CommandXboxController(0);
}
```

The `CommandXboxController` have extra features that we'll be using to streamline code, and access buttons by name (like `.x()` and .y() ) which don't get with other joystick variants like CommandJoystick or just Joystick.

Now that we have our joystick, we have several ways to interact with it, but we don't have the code mechanics to do so yet. We need to understand two important topics


# Switching behaviours with Commands

A `Command` is a structure that represents a behavior of a mechanism, group of mechanisms, or a whole robot. More formally, we might call this a "state", or "configuration". 

During operation, Commands make it easy to change between different behaviors, and to do so in a fairly structured, consistent way.

Building on our simple motor example from [[Basic Motor Control]], we can now improve our control process.

We previously controlled the motor using a simple method of running it in a Periodic method, that was called every robot loop

```java
	public void periodic(){
		motor.set(0.2); 
	}
```

and found that this wasn't easy to manipulate. As written, we cannot change the actual bot behavior without changing the code. So, let's move that useful operation (setting the motor) and convert it to a few different Commands

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	
	public void periodic(){
		//Remove our motor call from here
	}
	
	public Command forward(){
		return run(()->{ motor.set(0.2); });
	}
	
	public Command backward(){
		return run(()->{ motor.set(-0.2); });
	}	
	
	public Command stop(){
		return run(()->{ motor.set(0); });
	}
}
```

### A bit of explanation and buildup
For now, let's focus on the simple Run command. This command takes a single function, and when that command is active, it runs that function every loop.

The strange syntax is called a "[[Lambdas|Lambda]]" or "Anonymous Function". It's just  like a typical function (you run the code inside when called, not when you encounter it in the file), but it has some special syntax to let you put that code inside other functions.

Our `run()` needs a function with no arguments. So, let's start with a a Lambda that did nothing which looks like`()->{}` ; However, we need a lambda that sets the motor, so we stuff in our  `motor.set(0.2)`, and get `()->{ motor.set(0.2); }`. 

We then take that whole Lambda, and pass it into our `run(...)` method. This gives us the final result seen above: `run(()->{ motor.set(0); });` . 

That `run(...)` function generates a new Command, and then lets us access it! Hooray!

However, to go one step further we generally we actually want nicer names to deal with in code. So, instead of just using `run(...)` directly (and building it everywhere we need to set the motor, we instead put it inside a function (forward, backward, or stop) and return it from there.

Now we can easily put `stop()` or `forward()` anywhere we want a new Command to represent that action. 

While a bit complicated, this type of function (a function that takes one thing and returns a newly built object) is called the [[Code Patterns|Factory Pattern]]. The `run(...)` is one of several factory methods we'll encounter to make commands.

# Subsystems (aka Mechanisms)


# Periodic condition checking with Triggers

`Triggers` are an object that represent a operation that returns true or false. These are useful because Triggers get checked automatically, and then we can do stuff when that condition changes. 