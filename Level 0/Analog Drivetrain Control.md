## Goals: 

- Control the drivetrain with joysticks
- Control the drivetrain with numerical functions.



# Using a Joystick for control


Adding a joystick, at long last, allows us to precisely control the robot. In order to do this, we need to slightly change our code structure (but we're equipped for it!)

We've observed that `differentialDrive.arcadeDrive(0.2,0);` is the key. We chose 0.2 arbitrarily, but this *could* be a value from our joystick. We want that! 

However, careful consideration shows the problem: Our `joystick` exists in RobotContainer.java, and we simply don't have access to it in our ExampleSubsystem where we've putting our drivetrain. We need to bridge that gap.

We can do this by creating a new command that takes a Joystick as an argument. This would look like this:

```java
public class ExampleSubsystem extends SubsystemBase{
	public Command driveWithJoystick(CommandXboxController joystick){
		return run( ()->{ differentialDrive.arcadeDrive(0.2,0); } );
	}
}
```

This gives this function access to the joystick, just like we do outside in RobotContainer. This is a particular technique we'll see a lot. 

Now we just need to actually read from it.
```java
public class ExampleSubsystem extends SubsystemBase{
	public Command driveWithJoystick(CommandXboxController joystick){
		return run( ()->{ differentialDrive.arcadeDrive(
			joystick.getLeftY(),
			joystick.getRightX()
		); } );
	}
}
```

This gives us the command structure we need, and we can add it to RobotContainer similarly to our other commands. We have to bind this command to a button, so we'll just pick `rightTrigger()`

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(joystick)
		);
	}
}
```

We can put this on the robot and drive! It works! Mostly! It drives backwards, and turns wrong. Not your fault! 

Joysticks have the convention that "up" is negative Y values, which we associate with "backward". 
Joysticks also have "right" as positive, which is great until you realize this is a "positive turn angle", and we've established "positive turn" is "counterclockwise".... or to the left.  AAAUGH!

In this case, both conventions are correct for their particular standards; Our motor is inverted correctly, and we can't change the joystick convention. But the _interaction_ between conventions is wrong. So, the best place to fix this where those two conventions meet. Just add a negative on each. 
```java
public class ExampleSubsystem extends SubsystemBase{
	public Command driveWithJoystick(CommandXboxController joystick){
		return run( ()->{ differentialDrive.arcadeDrive(
			-joystick.getLeftY(),
			-joystick.getRightX()
		); } );
	}
}
```

We're finally approaching the final piece of this puzzle. We want the joystick to work _without_ hitting the right bumper. 

This is actually pretty straightforward: Subsystems (Mechanisms) have the notion of a "default command", which is what it does when nothing else is running. 

This is pretty straightforward: Instead of binding our drive command to a joystick, we simply say "use this as default".  

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		exampleSubsystem.setDefaultCommand(
			exampleSubsystem.arcadeDrive(joystick)
		);
	}
}
```


`[!!alert-triangle|Note|var(--color-yellow-rgb)]` This section may change during the competition season due to major changes on the software side after Kickoff. 2027 may not have default commands and require other ways to manage this.




# Using externally provided values

Passing in a joystick is handy for manual driving, but it's *not* great if we want to generate these values some other way. A common use case is turning toward a [[Gyro Sensing|Gyro Heading]] heading, or using some other sensor like vision, or just as part of our autos. We'll examine these in detail later. 

For now, instead of using a joystick, we'll need to figure out how to pass in external values, allowing us to meet these needs.

# Re-examining our base function

Our `differentialDrive.arcadeDrive` function is *so close* to what we want. 

For now, we might try what we did with the joystick control, and observe what happens. Let's add parameters for the `forward` and `turn` values, as we did before.

```java
public class ExampleSubsystem extends SubsystemBase{
	//A "double" just represents a decimal point value, like 1.0, 0.2, or 3.14
	public Command arcadeDrive(double forward, double turn){
		return run( ()->{ differentialDrive.arcadeDrive(forward,turn); } );
	}
}
```

We can then move the values into RobotContainer, and using a our custom values on the right trigger

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other drivetrain stuff we've made ....

		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(0.2,0)
		);
	}
}
```

At the moment, both ways of writing this are *basically* the same: We just changed where we moved the values. But this allows us a greater degree of freedom from just forward, backward, left right. This works! 

So, let's try reading from our joystick, and using those values. 

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		//Note we're negating the values like we did before
		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(
				-joystick.getLeftY(),
				-joystick.getLeftX())
			)
		);
	}
}
```

Go ahead and try it! Using the right trigger will enable your new joystick drive mode. It's very likely to be not exciting, for reasons that are not obvious to new programmers.

The reason is that when the code *first* starts, it sees `joystick.getLeftY()`, and reads the joystick. It's probably centered, so the value of the Y axis is zero. same with `joystick.getLeftX()`, also zero. 

Which means, this is basically a more complex way to write  
```java
		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(0,0)
		);
```

So, instead of passing values, let's instead pass in a _function_. The function can describe "here's how to get the value from the joystick". 

If your first thought was "We can do this with Lambas again!" you're paying very good attention. 

```java
public class ExampleSubsystem extends SubsystemBase{
	public Command arcadeDrive(
	    //Instead of Passing in the value, we pass in how to get the value
		DoubleSupplier forwardSupplier, 
		DoubleSupplier turnSupplier
	){
		return run( ()->{ differentialDrive.arcadeDrive(
			forwardSupplier.getAsDouble(),
			turnSupplier.getAsDouble()
		); } );
	}
}
```

We also appended `.getAsDouble()` to where we read the value. This method runs the DoubleSupplier lambda, and returns whatever the value it calculates.

We'll see this causes an unhappy compiler warning in RobotContainer, so let's fix that next.
```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(
				//Just add the ()-> here! 
				()->-joystick.getLeftY(),
				()->-joystick.getLeftX())
			)
		);
	}
}
```

There we go! We now have a right trigger that behaves *exactly* like the version where we passed in a joystick. 

# Stopping on end
For the sake of example clarity, you might notice we fell back to not having the "stop the motor" when exiting bit in these commands! Let's add that back in. 
```java
public class ExampleSubsystem extends SubsystemBase{
	public Command arcadeDrive(
		DoubleSupplier forwardSupplier, 
		DoubleSupplier turnSupplier
	){
		return run( ()->{ differentialDrive.arcadeDrive(
			forwardSupplier.getAsDouble(),
			turnSupplier.getAsDouble()
		); } )
		.finallyDo(()->differentialDrive.arcadeDrive(0,0))
		;
	}
}
```
In this case, I've added it using `.finallyDo()` instead of swapping our `run` to `runEnd()`. In this case, it's just easier when retrofitting, and it helps reduce some syntax complexity (we're getting a lot of mess in there! ).

# Cleaning Up The Drivetrain Code

After all this work, you probably have noticed there's a *lot* of copy-pasting the same basic code, and fiddling with a few numbers. It's time for a bit of cleanup, or what is commonly called a "refactor".

The goal here is to look at our code, and try to minimize some of this copy-paste and excess cruft that we've built up. Here's what we have now.

```java
public class ExampleSubsystem extends SubsystemBase{
	//4 single function commands
	//These all have the same code with different numbers
	public Command forward(){...}
	public Command reverse(){...}
	public Command left(){...}
	public Command right(){...}
	//Our joystick option, which is the same code, but with a joystick read
	public Command driveWithJoystick(CommandXboxController joystick){...}
	//Mostly the same code, but with our Supplier functions
	public Command arcadeDrive(DoubleSupplier forward, DoubleSupplier turn){...}
	}
```

Each and every one has the same overall structure, and ultimately, just boils down to  
```java
public Command functionName(maybe){
	return runEnd(/*stuff*/ drivetrain.arcadeDrive(/*values*/)/*stuff*/)
}
```
which is a *lot* of code syntax for what we're trying to do.

We can replace a lot of this code structure by using our most generic variation of this operation, and using that as our building block. For example, we can replace `Forward` with this. 

```java
public Command forward(){
	return arcadeDrive(()->0.2,()->0);
}
```

That's so much less code for the same result. Reverse, left, and right can all be stripped down to this simpler notation with almost no effort, and give us these named, simple commands.

We *could* replace `driveWithJoystick(joystick)` command with our "read the joystick in RobotContainer" version, but having a dedicated command like this is fine too. It gives a good place to make changes specific to the driver's preferences later. We can still clean it up though!

```java
public Command driveWithJoystick(CommandXboxController joystick){
	return arcadeDrive(
		()-> -joystick.getLeftY(),
		()-> -joystick.getLeftX()
	);
}
```

Now we really only have one "messy" command function, which is great! 

You'll also note, in this process we also inadvertently ensured that *all* our commands get the "stop on exit" for free! `arcadeDrive(...)` does this, and everything else uses, it so we've greatly reduced the chance of forgetting this as we add commands later.

This kind of thing will happen a lot with robots! If you find yourself doing copy-paste versions of functions or commands, consider cleaning up with this type of "building block" function. 

Most of the time, you'll only catch this type of "cleanup"  into very clean code after you've made a mess Or, sometimes, you'll feel like there's a better way, but you don't know how to do it. That's fine! Make a mess for now, get things working, and then ask how to improve it. 