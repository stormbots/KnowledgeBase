

# Voltage vs Set


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

