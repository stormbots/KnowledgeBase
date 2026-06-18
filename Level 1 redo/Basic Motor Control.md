### Goals
- Create a Motor
- Set up the drivetrain with Spark Maxes
- Establish current limits and configs
- Open Loop Control



# Creating a Motor

To create a motor, you would create an appropriate motor Object.  Motors generally should be declared inside a subsystem. For now, we'll use ExampleSubsystem.java as part of the created template.

```java
public class ExampleSubsystem extends SubsystemBase{
	//The first parameter is the motor's ID, and unique to each motor.
	// Ther second is the type of motor; We only use Brushless.
	SparkMax motor = new SparkMax(1, MotorType.kBrushless);
}
```

Note the "type" of the motor is a SparkMax ; This is the type of motor controller we're talking to. For the most part, we'll use "Spark Max" interchangeably with "motor".

You'll likely need to find your motor's ID using the  [[Rev Hardware Client]] . The Hardware Client allows you to change it to prevent overlaps, or change motor settings directly.
# Spinning a motor

The easiest interaction with a motor is to directly set the output.

The code for this should be put somewhere that executes every loop. For simplicity, let's add it to our Periodic loop.

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(1, MotorType.kBrushless);
	
	public void periodic(){
		//Set the motor's output to 20% of it's capability
		motor.set(0.2); 
		//This range is limited to -1 to 1, representing the percentage 
		// of output power and a direction.
		// Larger values will be constrained to -1 to 1 automatically.
		//This is often called the "throttle" or "percent output"
	}
}
```


Generally, we also need to ensure the Subsystem containing our motor exists.  In our example, we'll see that this example is in RobotContainer.java , so we're covered.
```java
public class RobotContainer{
	public ExampleSubsystem exampleSubsystem = new ExampleSubsystem();
	
	//... Lots of  other stuff down here
}
```

You now have a spinning motor! 

Note, it runs when the robot is enabled, and can't be _stopped_ without disabling the robot. This makes this way of controlling a motor fairly useless outside of tutorials, but we'll fix this soon using Commands.

# Configuring a motor

Motors are configured using a Config object. This config can be saved and reused, but in general you only need to do so once per motor, so it's often in a constructor for your subsystem. 

There's a lot of settings that we'll encounter for more complex robots, but for now there's just a couple basics we'll need right away.

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(1, MotorType.kBrushless);
	
	public ExampleSubsystem(){
		//This creats a new motor config
		var config = new SparkMaxConfig();
		
		//Set the motor to a lower maximum power output.
		//A proper motor limit makes them safer to operate.
		config.smartCurrentLimit(10);
		
		//This lets you switch which direction the motor spins 
		// when you tell it to "go forward".
		//We'll form opinions on "correct directions" later!
		config.inverted(false);
		
		//Controls whether the motor tries resist or allow external motion
		// IdleMode.kBrake will resist motion 
		// IdleMode.kCoast will not
		config.idleMode(IdleMode.kBrake) 
		
		//Now, we can Apply our config to the motor
		motor.configure(
			config,
			ResetMode.kNoResetSafeParameters,
			PersistMode.kNoPersistParameters
		);
	}
	
	public void periodic(){
		motor.set(0.2); 
	}
}
```

For now, you can change `config.inverted(false)` to `config.inverted(true)` and see the motor switch directions on enable. 

You can also set `idleMode(Idlemode.kCoast)`, and when the bot is disabled, you'll see that the mechanism is much easier to turn.
