
### Goals:
- Provide human input to the robot
- Have a fully controllable chassis


# Coding up a controller

Most example projects have a controller. In the ones we'll use, it's usually in `RobotContainer.java`.

There's a few different ways to construct controller objects, but the common one is `CommandXBoxController`

```java
public class RobotContainer{
	private final CommandXboxController joystick = new CommandXboxController(0);
}
```

The `CommandXboxController` have extra features that we'll be using to streamline code, and access buttons by name (like `.x()` and `.y()`) which aren't available on other joystick variants like `CommandJoystick` or `Joystick`

Now that we have our joystick, we have several ways to interact with it, but we don't have the code mechanics to do so yet. We need to understand two important topics


> [!NOTE] Joystick naming
> Sample projects start with a joystick created, using an awkward name
> `private final CommandXboxController m_driverController = ...`
> which is difficult to use in examples.
> You may want to change the name of this variable and fixing the other places it's used before moving forward with examples


# Switching behaviours with Commands

A `Command` is a code structure that represents a behavior or action. This can be of a single mechanism (like the drivetrain, an intake, or roller) or of the whole robot. It's best to think of these as the "verbs" of the robot. You "Command" it to do something, and it does that until it's time to stop.

Using Commands, we can adjust our example from  [[Basic Motor Control]], into a set of Commands that better describe what we want the chassis to do.

We previously had this:

```java
	public void periodic(){
		differentialDrive.arcadeDrive(0.2,0); 
	}
```

and found that this wasn't easy to manipulate. As written, we cannot change the actual bot behavior without changing the code. So, let's move that useful operation (setting the motor) and convert it to a few different Commands

```java
public class ExampleSubsystem extends SubsystemBase{
	
	public void periodic(){
		//Remove our differential drive call from here
	}
	
	public Command forward(){
		return run( ()->{ 
			differentialDrive.arcadeDrive(0.2,0); 
		} );
	}
	
	public Command backward(){
		return run( ()->{ 
			differentialDrive.arcadeDrive(-0.2,0); 
		} );
	}	
	
	public Command stop(){
		return run( ()->{ 
			differentialDrive.arcadeDrive(0,0); 
		} );
	}
}
```

### Explaining the syntax
For now, let's focus on the simple Run command. This Command type runs the provided code every loop while the command is active. 

The `run(...)` command is one of several helper functions for building Commands. We'll talk more about these when looking at sequencing and building autos.

The strange syntax is called a "[[Lambdas|Lambda]]" or "Anonymous Function". It's just  like a typical function (you run the code inside when the function/lambda called, not when you encounter it in the file), but it has some special syntax to let you put that code inside other functions.

Our `run(...)` needs a function (or Lambda) that  accepts no arguments. So, let's start with a Lambda that did nothing which looks like`()->{};` . Now we have the structure we need, so we can stuff in our motor command `differentialDrive.arcadeDrive(0.2,0)`, and get `()->{ differentialDrive.arcadeDrive(0.2,0); }`. 

Now we have a Lambda that does what we want, and shove it into our `run(...)` method. This gives us the final result seen above: `run(()->{ differentialDrive.arcadeDrive(0.2,0); });` . 

Having given our `run(...)` function the right input, it now that generates a new Command, and then lets us access it! Hooray!

However, to go one step further we generally we actually want nicer names to deal with in code. So, instead of just using `run(...)` directly (and building it everywhere we need to set the motor, we instead put it inside a function (forward, backward, or stop) and return it from there.

Now we can easily put `stop()` or `forward()` anywhere we want a new Command to represent that action. 

While a bit complicated, this type of function (a function that takes one thing and returns a newly built object) is called the [[Code Patterns|Factory Pattern]]. The `run(...)` is one of several factory methods we'll encounter to make commands. 

# Subsystems (aka Mechanisms)

A Subsystem represents an actuator or actuator group, or simply "a mechanism". We often use it to separate out code for that mechanism so it's a bit easier to deal with.

However, Commands also interact with Subsystems using the `requirements` conventions. A subsystem can't do two things at once, so it often doesn't make sense to have two Commands at once. When a new Command starts, old commands stop running.

Breaking down mechanical parts of the bot into "subsystems" is not relevant yet, but we'll come back to that.

`[!!flame|2027 Season Code|var(--color-cyan-rgb)]` will be renaming Subsystems as Mechanisms , as it better describes the intent of that code structure.

# Periodic condition checking with Triggers

`Triggers` represent some process that returns a true or false value. These are useful because Triggers get checked automatically, and make it easy to act on specific values, or changes in those values. We'll discuss them more later; They're often used for sensors.

For now, the only Triggers we care about are the controller buttons. We want to know "is it pressed", and `CommandXboxController()` provides these triggers for us with handy names.

```java
joystick.x() //Trigger that represents X being pressed
joystick.y() // trigger that represents Y being pressed
// And so on
```

For now the most useful parts of Triggers are these two:
```java
// Runs a command while a button is held,
// and stops the command when the button is released
joystick.x().whileTrue(/**/)

// Runs a command when a button is pressed, but never stops it.
joystick.y().onTrue(/**/) 
```

# Controlling the robot

At last, we have all the parts in place to actually operate the robot via buttons. 

```java
// In RobotContainer.java
public class RobotContainer{
	// This function is called by the consturctor, Robotcontainer().
	// It's just here  to  help us organize our button assignments.
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		joystick.a().whileTrue(exampleSubsystem.backward());
	}
}
```

We can upload this new code to the robot, and we'll see the expected result! Go ahead, do it! 

# OK but this is complicated I just wanted to drive the robot?

First, that's not a question. 

But it's because robots are complicated! The "simple" and easy to explain way to grab a joystick (using functions like `periodic`) to a robot *will* work, but not for long. 

As you try to get more out of your robot, you'll quickly find the "simple" way stops working well, and your code will quickly spiral into an unmanageable mess. It doesn't take a lot of buttons before the "simple" way becomes very difficult to work with, debug, and reason about. Especially with multiple people involved.

In FRC, Commands tend to be a good solution for these (and other) problems, and give us a good way to handle it. You can only run one command at a time, and most commands are just a few simple lines of code, so debugging can be straightforward.

The hard part, instead, is figuring out how all the parts are glued together. It's tricky now, but will make perfect sense after a bit of practice. 

# More Buttons!

We now have our Forward and Backward, so make `left()` and `right()` command. They'll  look very similar to the examples above, but will be left as an exercise for you. If you get stuck, ask a mentor or partner!

Once those are done, connect them to the other buttons using  `joystick.b()` and `joystick.x()`. Again, very similar. 

You should now be able to drive the robot around the room using the buttons. 

# But I want a joystick! 

That's up next! See [[Analog Drivetrain Control]].

