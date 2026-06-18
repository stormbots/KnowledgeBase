### Goals
- Create a Motor
- Establish current limits
- Open Loop Control



# Creating a motor

To create a motor, you would create an appropriate motor object. Motors generally should be declared inside a subsystem. 

```java
public class ExampleSubsystem extends SubsystemBase{
	//The first parameter is the motor's ID, and unique to each motor.
	// Ther second is the type of motor; We only use Brushless.
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
}
```

You'll likely need to find your motor's ID using the  [[Rev Hardware Client]] . This also allows you to change it to prevent overlaps, or to get the motor into an established configuration.
# Spinning a motor

The easiest interaction with a motor is to directly set the output.

The code for this should be put somewhere that executes every loop. For simplicity, let's add it to our Periodic loop.

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	
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


We also need to tell our robot that this subsystem (consisting of a single motor) exists. This might be done in some examples.
```java
public class RobotContainer{
	public ExampleSubsystem exampleSubsystem = new ExampleSubsystem();
	
	//... Lots of  other stuff down here
}
```

You now have a spinning motor! 

Note, it runs when the robot is enabled, and can't be _stopped_ without disabling the robot. This makes this way of controlling a motor fairly useless outside of tutorials, but we'll fix this shortly with Commands.

# Configuring a motor

Motors are configured using a Config object. This config can be saved and reused, but in general you only need to do so once, so it's often in a constructor. 

There's a lot of settings that will be touched on over time, but for now there's just a couple basics seen on almost every motor

```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	
	public ExampleSubsystem(){
		//This creats a new motor config
		var config = new SparkMaxConfig();
		
		//Set the motor to a lower maximum power output.
		//A proper motor limit makes them safer to operate.
		config.smartCurrentLimit(10);
		
		//This lets you switch the "Forward" direction of a motor
		//We'll form opinions on "correct directions" later
		config.inverted(false);
		
		//Controls whether the motor tries resist or allow external motion
		// IdleMode.kBrake will resist motion 
		// IdleMode.kCoast will not
		config.idleMode(IdleMode.kBrake) 
		
		//Apply our config
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

# Other Common Motor Interactions

### Voltage Control

Instead of directly setting a "percent of maximum power" using `.set(...)` you can provide a voltage.
```java
public class ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	
	public void periodic(){
		//Set the motor's output to 2 volts
		motor.setVoltage(2); 
	}
}
```

The advantage of direct voltage control is predictability and stability over a match. This is due to the behaviour of the battery itself. As the battery is drained, the voltage it provides decreases. 

When setting the throttle/percentage using `.set(...)` the "provided output" decreases in proportion to the  battery drain! This means things that "work well" at the start of a match might be less effective at the end of a match.

In contrast, setting the voltage directly ensures the output doesn't change based on the battery,
![[voltage-vs-percent.svg]]

Note, that setVoltage *still* can't provide more voltage than is physically available!

If you like using `.set(...)` you can also also use the Voltage Compensation option on your motor controller. 
```java
	//... when configuring your motor
	var config = new SparkMaxConfig();
	config.voltageCompensation(10) // We've now declared 100% to be 10 volts
	//... and when setting it
	motor.set(0.75) // Now is 7.5 volts!
```

