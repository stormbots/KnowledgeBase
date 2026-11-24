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
			joystick.getLeftX()
		); } );
	}
}
```

This gives us the command structure we need, and we can add it to RobotContainer similarly to our other commands. We have to bind this command to a button, so for now let's just pick `rightTrigger()`

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		joystick.rightTrigger().whileTrue(
			driveWithJoystick.arcadeDrive(joystick)
		);
	}
}
```

We can put this on the robot and drive! It works! Mostly! You have to hold a buttton to drive it, and it drives backwards, and turns wrong. Not your fault! 

Joysticks have the convention that "up" is negative Y values, which we associate with "backward". 
Joysticks also have "right" as positive, which is great until you realize this is a "positive turn angle", and we've established "positive turn" is "counterclockwise".... or to the left.  AAAUGH!

In this case, both conventions are correct for their particular standards; Our motor is inverted correctly, and we can't change the joystick convention. But the _interaction_ between conventions is wrong. So, the best place to fix this where those two conventions meet. Just add a negative on each. 
```java
public class ExampleSubsystem extends SubsystemBase{
	public Command driveWithJoystick(CommandXboxController joystick){
		return run( ()->{ differentialDrive.arcadeDrive(
			-joystick.getLeftY(),
			-joystick.getLeftX()
		); } );
	}
}
```

We're finally approaching the final piece of this puzzle. We want the joystick to work _without_ hitting the right bumper. 

This is actually pretty straightforward: Subsystems (Mechanisms) have the notion of a "default command", which is what it does when nothing else is running. 

This is pretty straightforward: Instead of binding our drive command to a joystick, we simply say "use this as default".  We can replace the button version with this one:

```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		exampleSubsystem.setDefaultCommand(
			exampleSubsystem.driveWithJoystick(joystick)
		);
	}
}
```

At long last, the joystick will just move the robot without  anything else. And, you can press the A/B/X/Y buttons to do specific other operations. 

Now is a good time to take a short break and play with the robot. 

# Using externally provided values

Passing in a joystick is handy for manual driving, but it's *not* great if we want to generate these values some other way. A common use case is turning toward a [[Gyro Sensing|Gyro Heading]] heading, or using some other sensor like vision, or just odd values as part of our autos. We'll examine all of these later. 

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
			); } 
		);
	}
}
```

We also appended `.getAsDouble()` to where we read the value. This method runs the DoubleSupplier lambda, and returns whatever the value it calculates.

We'll see this causes an unhappy compiler warning in RobotContainer, so let's fix that next by adding our `()->` syntax
```java
public class RobotContainer{
	private void configureBindings() {
		joystick.y().whileTrue(exampleSubsystem.forward());
		//... other buttons we've made ....

		joystick.rightTrigger().whileTrue(
			exampleSubsystem.arcadeDrive(
				//Just add the ()-> here! 
				()-> -joystick.getLeftY(),
				()-> -joystick.getLeftX())
			)
		);
	}
}
```

There we go! We now have a right trigger that behaves *exactly* like the version where we passed in a joystick. 

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
	return arcadeDrive(()->0.2, ()->0);
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

# Renaming ExampleSubsystem to Drivetrain

You may have noticed that we've just been using ExampleSubsystem to house our Drivetrain code. Let's fix this! 

Your first thought might be to 
- Go to `ExampleSubsystem.java`
- Select the bit where it says `ExampleSubsystem`
- And change it
Which will end poorly; This will cause a *lot* of mismatches in your code, as Java can no longer find the ExampleSubsystem class that other files (like RobotContainer.java) are looking for.
The same happens with other obvious approaches like changing the filename. 

You *can* simply run around fixing all the compiler errors, but there's a better way. 

The best way to do this is using VS Code's "Rename" option; This little tool keeps track of all the places in your code that reference this Class, and fix it all for you automatically!

This is *usually* available by just right clicking on the thing you want to rename, and find Rename Symbol. 
![[rename-1-menu.png]]

If it's *not*  on the right click menu, or to feel extra pro, you can use the F2 key (which is usually "Rename" in most software). Some laptops default to having F2 work as a volume key, in which case Fn+F2 will do it. 

This will bring up a small text box for the new name. Enter Drivetrain, and then hit Enter!
![[rename-2-input-field.png]]

This will rename the class, change any imports or references, and rename the file itself (which Java projects require). 

Future examples will use `Drivetrain` (note the capitalization!), so that's the recommended name for now. 

Of note, this _does not_ change the name used by instances of this class. If we go to RobotContainer, we'll see this:
![[rename-3-incomplete.png]]

Just rename `m_exampleSubsystem` to `drivetrain` (again, note capitalization!)

![[rename-4-complete.png]]

Now, your drivetrain is properly named.
