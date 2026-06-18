
### Goals
- Create a second Motor+Config
- Set up the drivetrain object
- Make a drivetrain go forward

# Differential Drives
A Kitbot (and our testbench bot, TABI) has two motors on either side. Together, those two motors work to move the robot, and allow it to turn. 

If you just spin one motor (as we've seen) then it rotates around the other. So, let's set up the second motor.

# Left+Right Motors

Currently we have one motor, called motor. But well need two so let's prepare for this

```java
public class ExampleSubsystem extends SubsystemBase{
	//The first parameter is the motor's ID, and unique to each motor.
	// Ther second is the type of motor; We only use Brushless.
	SparkMax motor = new SparkMax(1, MotorType.kBrushless);
}
```

Let's rename this motor to `motorLeft`. This will cause a error in your `ExampleSubystem` and `periodic` functions  too, so you'll need to fix it there as well. 

This is also a case where VS Code can help. If you right click the variable name (`motor`) you can use `Rename Symbol` and enter the new name. This will fix all places where this specific `motor` is being used, and you can avoid many such errors . 

Now we can just create a second motor called `motorRight`, and we'll have this:

```java
public class ExampleSubsystem extends SubsystemBase{
	//The first parameter is the motor's ID, and unique to each motor.
	// Ther second is the type of motor; We only use Brushless.
	SparkMax motorLeft = new SparkMax(1, MotorType.kBrushless);
	SparkMax motorRight = new SparkMax(4, MotorType.kBrushless);
}
```


Just for fun, let's add this to our Periodic function.

```java
public class ExampleSubsystem extends SubsystemBase{
	// ... there's other stuff up here
	
	public void periodic(){
		motorLeft.set(0.2);
		motorRight.set(0.2); 
	}
}
```

Enable your robot, and see what happens! 

In most setups, I expect this will spin in place! Probably fairly quickly!
# Making The Drivetrain go "Forward"

Since you gave both motors a "positive" value, you *probably* wanted the robot as a whole to go forward. But, looking at the robot mechanically, you'll see the issue. 

A "positive" output causes the motor itself to spin "clockwise" in a consistent way. But the left and right side are "mirrored" relative to eachother, so the impact on the robot's motion is flipped! 

You *might* be tempted to just change one motor to "negative" and fix  this. However, you have then remember _which_ motor gets a negative, every time you want to deal with this. It's much better to tell the motor "hey, you're mounted the wrong way" and have it take care of this automatically. Let's do that.
# Configuring your other motor

We previously configured a motor, which looks like this when we strip out the commentary, which configures motorLeft, but _not_ motorRight. 

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motorLeft = new SparkMax(1, MotorType.kBrushless);	
	SparkMax motorRight = new SparkMax(4, MotorType.kBrushless);
	
	public ExampleSubsystem(){

		var config = new SparkMaxConfig();
		config.smartCurrentLimit(10);
		config.inverted(false);
		config.idleMode(IdleMode.kBrake);

		motorLeft.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);
		
	}
	
	// ... periodic is down here somewhere
}
```

We can actually just fix this fairly easily: Just apply the same config to the second motor!

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motorLeft = new SparkMax(1, MotorType.kBrushless);	
	SparkMax motorRight = new SparkMax(4, MotorType.kBrushless);
	
	public ExampleSubsystem(){

		var config = new SparkMaxConfig();
		config.smartCurrentLimit(10);
		config.inverted(false);
		config.idleMode(IdleMode.kBrake);

		motorLeft.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);

		motorRight.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);
	}
	
	// ... periodic is down here somewhere
}
```

But this won't fix our  problem. If we set `config.inverted(false)` it spins how it is now. 

But if we set `config.inverted(true)` it just spins the _other_ way.

Instead, what we need to do is this:

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motorLeft = new SparkMax(1, MotorType.kBrushless);	
	SparkMax motorRight = new SparkMax(4, MotorType.kBrushless);
	
	public ExampleSubsystem(){

		var config = new SparkMaxConfig();
		config.smartCurrentLimit(10);
		config.inverted(false); 
		config.idleMode(IdleMode.kBrake);

		motorLeft.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);
		// Left will now be configured with inverted(false)
		
		// Now, we can change the config before sending it to right.
		config.inverted(true); 

		motorRight.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);
		//Right is now inverted relative to left.
	}
	
	// ... periodic is down here somewhere
}
```

If your robot goes "backwards", just flip the inversion that each motor sees, and you'll be set. 

Now your robot will drive straight, with "positive" meaning "forward" for each side. 

As a thought process, think about what you'd need to send to each motor to make it 
- spin in place clockwise
- reverse
- drive in a big circle
- drive in a small circle
- pivot around the left wheel

While we can write a bot while thinking about motors like this, there's an easier way.
# Streamlining the Driving Process 

Now, let's start changing our thinking from "two motors" to "one drivetrain". We'll do this with the help of a `DifferentialDrive` object.

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motorLeft = new SparkMax(1, MotorType.kBrushless);	
	SparkMax motorRight = new SparkMax(4, MotorType.kBrushless);
	
	DifferentialDrive differentialDrive = new DifferentialDrive(
		motorLeft, 
		motorRight
	);
	
	// ... constructor and periodic not shown
}
```

The, let's go to our periodic and remove the existing `.set(...)`  calls, and replace with a single DifferentialDrive call that does the same thing:

```java
public class ExampleSubsystem extends SubsystemBase{
	public void periodic(){
		differentialDrive.arcadeDrive( 0.2, 0.0 );
	}
}
```

`arcadeDrive` uses a "forward" value and a "turn" value, and figures out what each motor should be doing. We just set the "forward" value to positive, and no turn value. 

```java
//Goes forward
differentialDrive.arcadeDrive( 0.2, 0.0 );
//goes backward
differentialDrive.arcadeDrive( -0.2, 0.0 );
//pivots to the left/counterclockwise
differentialDrive.arcadeDrive( 0.0, 0.2 );
//Drives in an circle
differentialDrive.arcadeDrive( 0.1, 0.2 );
```

This doesn't seem like it'll simplify things yet, but once we hook up a controller this will become clear. 

It's sometimes helpful to note that "left" is counterclockwise for math reasons, and follows mathematical conventions. A "more positive" rotation is counterclockwise, both for our robot and when graphing angles.

![[axes.svg]]
