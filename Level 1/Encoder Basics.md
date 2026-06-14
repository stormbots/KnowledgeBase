---
aliases:
  - Relative Encoder
tags:
  - sensor
---

Requires:
[[Robot Code Basics]]

## Success Criteria
- [ ] Configure a Spark motor encoder
- [ ] Read an encoder position
- [ ] Configure encoder range/units through gearing
- [ ] ~~Enable/Disable Soft Limits~~

## Synopsis
An Encoder is a sensor that provides information about a mechanism's rotation. 

Since motors generally apply power through rotation, clever use of encoders allows you to infer a significant amount of system information. 

Encoders wind up being one of our primary sensors on FRC bots, due to their prevalence and flexibility.

## How Encoders Work

#### Speed sensing with GearTooth encoders

The simplest version of an encoder is simple sensor that switches between one and zero during a rotation.

![[Incremental_encoder.gif]]

This simplified encoder is often called a "gear tooth" sensor, since it's commonly associated with sensors that directly measure the one or more teeth on a gear directly. This generates a single pulse train like this:
![[Digital_Gear_Tooth_Sensor.gif]]

This pulse train switching between 0 and 1 provides *some* information about the rotation of the shaft: By counting the time between changes you can calculate the speed of the system. By counting the number of cycles, you can also tell how many teeth have passed.

However, this sensor is limited: The pulse train looks exactly the same whether you're going forward or backward! So you can't know what direction you're going, just how fast or how far. 

As a result, this sensor type is mostly used for measuring velocity where the direction is known by some other mean: Often the output of a motor itself. 

### Quadrature Encoder

An effective way to add the direction back in is to just track a different position. By carefully selecting a location that triggers slightly off from the first one, you get some interesting effects.

![[Incremental_directional_encoder.gif]]

Tracking the signals (from left to right), you get signal values that look like this:

![[quad-encoder-waveform-output.png]]

Because of the way the two signals overlap, you now have 4 signal states: `00`, `01`, `11`, and `00`. And importantly, you from any particular state, you can only go to 2 of other 3 states! For example, you cannot go directly from `11` to `00` : You *must* go to either `10` or `01`. 

Because of this, you always know what way a rotation is happening: If you're at `11`, and go to `10`, you're going clockwise. 

This property means that quadrature encoders are much more capable than GearTooth encoders. It can still track velocity, but can measure velocities at low speeds, or with external inputs where you wouldn't know the direction. 

In addition, since you always know the direction, you can also just count the pulses as they come by. This lets you know precisely _how far_ something has rotated, effectively indefinitely. 

It's important to note that another name for these is a "Relative encoder", since it only tells you a rotation relative to when it started counting! This means where your system is when the sensor starts is an important consideration!

## Java Code 

#### The WPILib Encoder classes 
The Rio can directly handle external quadrature encoders using the [WPI Lib Encoders](https://docs.wpilib.org/en/stable/docs/software/hardware-apis/sensors/encoders-software.html) classes. While uncommon in modern FRC, they show up on occasion. 

#### Spark Encoders
Modern FRC uses brushless motors for most operations, and have built in encoders that are ready to use! As a result, this is the most common interaction with encoders.

These are accessed by interfacing with a Spark Motor controller, like so. 

```java
ExampleSubsystem extends SubsystemBase{
	//Note: Your motor ID will probably be something different!
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	RelativeEncoder encoder = motor.getEncoder();

	// ... other stuff this way
}
```


> [!NOTE] Rookie Coder notes
> If you're still getting the hang of code structure, you can work in the `ExampleSubsystem.java` file for now! It's all set up on new `Command Based Robot` templates, and will work like you're expecting.


> [!NOTE] Finding Motor IDs
> If you're unsure about the motor ID, you can do get it using the [[Rev Hardware Client#Identifying a motor ID]]
> 


We now have an encoder.... but it's not *doing* anything. Let's make it do something. Encoders are good at telling you how far they rotated, so let's print that. 

```java
ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	RelativeEncoder encoder = motor.getEncoder();
	
	// ... other stuff
	
	public void periodic (){
		System.out.println(encoder.getPosition());
	}
}
```

Deploy this to the robot, and you should pop up with the Riolog tab in VSCode. If not, hit CTRL+Shift+P to bring up the VS Code Command Pallete. Type "Start Riolog" to select the option, then hit enter.

You should now see a screen full of numbers. So... try rotating your motor. It should change, but it's unclear how much. We haven't configured it to any sort of a known state.

To get better diagnostic information such as graphs, or multiple values at once, you might check out [[Basic Telemetry]]

#### Configuring Spark Encoders

The [[Rev Hardware Client]] page can assist with adjusting parameters, which is useful for quick diagnostics and testing. 

To configure it in code, we need to do a few things. First, is set up a config object, and clear factory defaults. This means we have a lot of potential quirks cleaned up instantly.

```java
ExampleSubsystem extends SubsystemBase{
	SparkMax motor = new SparkMax(10, MotorType.kBrushless);
	RelativeEncoder encoder = motor.getEncoder();
	// ... other stuff
	public void ExampleSubsystem(){
		SparkBaseConfig config = new SparkMaxConfig();
		armMotor.configure(
			//The configuration settings to apply
			config,
			//Whether or not to reset all unspecivied values
			ResetMode.kResetSafeParameters,
			//Whether or not to keep the new values after 
			//power cycling the motor controller
			PersistMode.kPersistParameters
		);
	}	
	// ... other stuff
}
```

If you run this on your test bench, you'll probably see the numbers change differently than they did prior. By default, the encoder is configured to provide "Rotations" from the `getPosition()` method. If you rotate the motor shaft exactly once, the encoder will return 1. 

If your testbench has some gearing involved, you'll get a different number multiplied by the gearing. For example, if you're on a Kit Of Parts drivetrain, it might have a gear ratio of 12.75:1. This means it takes 12.75 motor rotations to rotate the output wheel once. Or, put the opposite way: If you rotate drivetrain wheel *exactly* once, you should get an encoder reading of 12.75. 

However, we generally care more about the _output_ of the system than the motor, so we want to apply a conversion factor so we can "read" the output directly. 

To get this, we need to take that logic above, and rotate our system's output exactly once. Record the number of "motor rotations" as your conversion factor, and we'll add it to the code.

```java
ExampleSubsystem extends SubsystemBase{
	// ... other stuff
	public void ExampleSubsystem(){
		SparkBaseConfig config = new SparkMaxConfig();
		var conversionfactor=12.75; //1 divided by whatever number you determined
		config.encoder.positionConversionFactor(1/conversionfactor);
		armMotor.configure(
			config,
			ResetMode.kResetSafeParameters,
			PersistMode.kPersistParameters
		);
	}	
	// ... other stuff
}
```

The "conversion factor" is just a multiplier from the base unit (1 rotation). Since we know that 1 output rotation generates 12.75 motor rotations, we want to divide by 12.75. 

And, after we deploy this, we'll see that we can rotate our wheel once, and get an "encoder" reading of 1. 

#### Velocity
We've been glossing over velocity, but it's time! This is helpful too. The process from this point is actually pretty easy, and we can get it with just a few minor changes. 

```java
ExampleSubsystem extends SubsystemBase{
	// ... other stuff
	public void ExampleSubsystem(){
		SparkBaseConfig config = new SparkMaxConfig();
		var conversionfactor=12.75; //1 divided by whatever number you determined
		config.encoder.positionConversionFactor(1/conversionfactor);
		//Configure the velocity conversion factor too
		config.encoder.velocityConversionFactor(1/conversionfactor/60);
		armMotor.configure(
			config,
			ResetMode.kResetSafeParameters,
			PersistMode.kPersistParameters
		);
	}	
	// ... other stuff
	public void periodic (){
		//Change to velocity
		System.out.println(encoder.getVelocity());
	}
}

```

The main detail of note is that we use the same conversion factor to account for gearing.
We also have two ways of doing velocity conversion, depending on our desired Rate unit. 

By default, `1/conversionfactor` will give us Rotations Per Minute. In this case, the extra `/60` is to convert from rev's default Velocity unit of Rotations Per Minute to Rotations Per Second. For many of our tasks, having the unit in "per second" winds up being more useful since it's easier to visualize.

Generally, we'll calculate the conversion factor using Position since it removes the aspect of time; This makes it much, much simpler to get accurate results. 

#### Rotation To Linear Distance

Now that we can convert motor rotations to output shaft rotation, we can go a step further and convert that output rotation to a linear motion. This is pretty straightforward: Over 1 rotation, you cover 1 circumference of distance. So, just divide your conversion factor by your wheel diameter. Since we're calculating "x distance per rotation", so you would expect a division here. 
The final configuration will look something like this:

```java
// ... stuff
	var circumference = Math::PI * 4; // 4 inch wheel
	//1 divided by whatever number you determined
	var conversionfactor=12.75/circumference;
// ... the rest of the stuff
```

#### Managing unit conversions

If the units to configure your encoder is overwhelming, consider [[Factor Label]] methods to help you step through it. This is a useful method of tabulating units to keep track of conversions like this. 

As a sample calculation, this is what it looks to set up the conversion for a KOP bot's drivetrain from motor rotations to inches per second.

| motor rev        | time       | gear                  | wheel             |     | output  |
| ---------------- | ---------- | --------------------- | ----------------- | --- | ------- |
| 1 motor rotation | 1 min      | 1 output rotation     | Pi * 4 inches     | =   | ?? inch |
| minute           | 60 seconds | 12.75 motor rotations | 1 output rotation |     | s       |

#### Empirical analysis
Another way we'll commonly handle encoder configuration is through a simple empirical process:
- Set the encoder conversion to 1 , putting the units as "motor rotations"
- Put the system to known physical condition that's easy to measure. (zero feet traveled, arm at horizontal, etc)
- Zero the encoder (either a power cycle or a code reset)
- Move the system to a new physical condition that's also easy to measure. (10 feet traveled, arm at vertical, etc)

Keeping with our Kitbot example, in factor label form it would look something like this:

| motor rev    | conversion          |     | Output     |
| ------------ | ------------------- | --- | ---------- |
| 538 rotation | ??? inches          | =   | 120 inches |
|              | ??? motor rotations |     |            |
Doing the math, we get $120/538$ inches/motor rotations, and our conversion factor is decided!

An advantage of this method is that it requires no knowledge about how a system is geared. This is great for getting a system running and validating calculations. It can also help with systems that have difficult to calculate ratios, such as winding ropes or straps. 

One big disadvantage is precision: Every time you repeat this calculation, the results are *slightly* different. This may or may not be a problem, depending on the mechanism. 

> [!BUG] Competition Consideration
> It's not advised to use empirical analysis for drivetrains! During competition, wheel wear will often force constant re-calibration. However, in pit environment it's very challenging to get space to push a drivetrain. This makes corrections time consuming and inaccurate, making your drivetrain and autos inconsistent.


## Practical Applications

Having insight into the robot provides a ton of value, especially when combined with  [[Triggers]] and [[Commands]] . This makes it easy to do things like 

- See if a shooter is near a desired RPM before feeding a game piece into it
- See how far you've driven and stop
- Check a system's speed to see if it jammed.

We're not quite ready to do things like precisely control RPM, speed, or distance. For that, we need a [[PID]].

## Limitations and problems

Quadrature encoders are powerful, but not without flaws. The main thing is fundamental to position tracking with this type of encoder:
- You need to already know where you are when you you *start* counting pulses to have a meaningful knowledge of the system's position
- If something causes it to lose count (such as loss of power or a code reset), then you've lost the connection to the reference, and now don't know where you are.

There's a couple ways to avoid this. The easiest way is to start at your reference, and not lose count! As you might expect, this places some constraints on how you as humans boot up the robot. It also carries a small (but non-zero) risk for the second case. 

For fully mitigating this, you need to make use of [[Homing Sequences]] , which allow you to fully recapture your reference from an unknown starting state.

Other options include simply using different sensors! For example, [[Absolute Encoders]] can capture rotation data. 


## To Reset or Not To Reset
Encoders all have functions to set or zero the current encoder value value. For our example, we could do this using the following code
```java
ExampleSubsystem extends SubsystemBase{
	// ... other stuff
	public void ExampleSubsystem(){
		//Reset encoder on boot ?
		encoder.setPosition(0);
	}
	// Other stuff
```

But *should* we? We need to consider both the mechanisms physical state and the code state carefully, since we expect them to correlate. Let's examine the typical points at which one or both of these can change

- When power is applied, the mechanism is in an unknown position, the encoder is at zero, and the code runs initialization.
- When you deploy new code after running it, the mechanism can be in any state, the encoder is unchanged (by default) , and our code re-runs initialization

This leads to the following cases: 
- If we power on the bot in an known reference state, the encoder will match it for the first run of code.
- If we do not reset position in initialization, we can always assert that the encoder still matches the physical position, even if we re-run or restart code.
- If we do reset position, we do so without being able to assert that we're at the hardware reference!  This means we have to physically check that we're at the hardware each time manually. 

Because of this, for now we probably *do not* want to zero the encoder, which allows us to set the physical state once on boot, and re-use it indefinitely. 

If needed, you can generate [[Homing Sequences]], which are ways for the bot to physically assert a known system state and do the reset encoders safely. These are usually optional (at least initially), but are critical for robust high-torque systems or high precision ones.