
## What's a Closed Loop Controller

Often in robotics, we care a lot about the final state of a system: The position it's at, the rate it's going, the height of an elevator, etc. 

However, the process of getting and maintaining this target state might not be trivial. It might require a variety of motor outputs to get there, change over time,  or be unpredictable due to external factors like game piece weight.

A closed loop controller is designed to manage exactly this. Instead of simply setting the output directly (a motor speed), we can instead provide a setpoint, representing the target state of our system. This can be a position, rate, temperature, brightness, or any other  measurable quantity. It's sometimes considered the "goal state".

A closed loop controller's job is to then manage the measured quantity and adjust the output to maintain that setpoint. In technical terms, we "feed back" the output as an input to this controller, forming a "closed loop" in our process diagram.

![[open-loop-closed-loop-diagram.png]]



## Controller Tradeoffs

Every controller style will have tradeoffs between many factors. Often times, these are 
- Precision: How well the controller tracks the setpoint
- Speed: How quickly the controller can get to the desired setpoint from typical states
- Stability: How well the controller reacts to unexpected situations. Some controllers *completely* break down and fail in certain conditions.
- Computational Efficiency: This affects resources used for maintaining control, and in some cases how fast you can run your closed loop system. In real world cases, faster is usually better.
- Tuning Difficulty: How hard it is to get the controller into a working, stable state. Some start working effectively in a few minutes; Others can 
- Conceptual Difficulty: Some controllers are easy to understand in an afternoon; Others require expert level understanding of complex mathematics to be able to work with effectively.
- Output capability: Some controllers are designed for output mechanisms that may not be available.


## PID
A [[PID]] is a general purpose, flexible, and powerful closed loop structure, that can "model" a wide variety of systems.

This is a most likely the closed loop option you want for typical FRC systems. 

## Bang Bang
as derived on the [[PID]] lecture, a Bang Bang controller looks at the error, and emits full output power depending on whether it's above or below. 

```java
class BangBang(){
	double max=1;
	double min=0;

	double calculate(double sensor, double setpoint){
		if(sensor < setpoint){ return max ; }
		else if(sensor >= setpoint){ return min; }
	}
}
```

This kind of controller tends to be sub-optimal on most systems, but is useful in situations where you have 
- Analog inputs
- Binary outputs
- Comparatively large inertia relative to system power. 

## Hysteresis

Hysteresis is a resistance to state changes, with numerous applications in physics and control theory, both advantageous and otherwise.

In control theory, hysteresis is often utilized intentionally to prevent rapid switching when a system is at setpoint, as shown in this modified bang-bang controller.

```java
class BangBangWithHysterisis(){
	double threshhold=1;
	double max=1;
	double min=0;
	double currentValue=0
	
	double calculate sensor, double setpoint){
		if(sensor < setpoint-threshhold){ currentValue=max;}
		else if(sensor >= setpoint+threshhold){ currentvalue=min;}
		return currentValue;
	}
}
```

In this code, you can see that if a setpoint is within 1 of the setpoint, it doesn't turn off _or_ turn on; It simply does what it was already doing.

This creates both slight overshoot and slight undershoot, but creates a range of acceptable values, which is often suitable for applications where a standard BangBang controller would be used.

This is something you'll commonly see on automatic heaters: As the room cools, they turn on when you're too cold, and only turn off once you're a bit too warm, and the cycle repeats. You're never quite _at_ the temperature you want, but generally pretty close to it. The hysteresis ensures that the heater always changes the temp by several degrees at once.

Hysteresis is also seen in automatic shifting algorithms in both cars and robots; This avoids unnecessary gear changes within ranges where either gearing is equally effective.

## Three-Point Controller

Similar to a Bang Bang (formally known as a two-point controller), a three-point controller has three states. Usually on/off/reverse, or off/low/high.
```java
class ThreePointController(){
	double threshhold=1;
	double max=1;
	double min=-1;
	double off=0
	double currentValue=0
	
	double calculate sensor, double setpoint){
		if(sensor < setpoint-threshhold){ currentValue=max;}
		else if(sensor >= setpoint+threshhold){ currentvalue=min;}
		return off;
	}
}
```

This looks a lot like the hysterisis control, but instead of overshooting through the middle zone, it simply turns off.

While not common in modern FRC, this control mode is very common in many real world applications due to the simple, cheap electronics needed to operate it.

In past FRC seasons, this type of control was used for relay controlled motors. It can also be used alongside two-pole pnuematic systems for somewhat analog control in air positioned systems. 

## Sqrt(err)

This is an odd, but useful technique, as much as a specific algorithm. By taking the square root of the error, then applying a gain, you get a sharp, aggressive response for small errors, but decreasingly aggressive as the error increases further.

```java
class SquareRootError(){
	double gain = 0.1;
	
	double calculate(double sensor, double setpoint,){
		var error = setpoint-sensor;
		output = gain * Math.signOf(error) * Math.sqrt(Math.abs(Error));
		return output;
	}
}
```

This simple algorithm tends to generate a "springy" response to disturbances, is simple to tune, and typically can hit the setpoint reliably under a wider variety of disturbance conditions.

This algorithm tends to work well for velocity systems like [[SuperStructure Rollers|Rollers]], [[Superstructure Shooter|Shooters]], or Drivetrain wheel speeds. When tuned effectively, disturbances will tend to generate the right amount of output to add momentum to the system, quickly getting things back to the velocity target.

It's best when performing continuous tracking of a setpoint, such as ones generated by position mapping from a joystick slider, or position/velocity targets generated by [[Motion Profiles]]. 

It can also provide satisfactory results for point-to-point motion using Swerve or Mechanum drivetrains using odometry-oriented control.

A notable flaw is that it has no meaningful compensation setpoint changes, and simply causes output spikes.

The major flaw of this algorithm stems from the tendency to result in high frequency oscillation at the setpoint, especially for systems with notable [[Mechanical Backlash|Backlash]]. Using a [[Slew Rate Limiting|Slew Rate Limiter]] on the output can improve response in these cases, often reducing the oscillation to negligible amounts. Allowing sharp oscillations make it sub-optimal for holding position in steady state conditions where there's no force or friction to keep it from mechanically thrashing gearing across backlash.


## Take Back Half
An integrator-oriented control loop with an output back-off when the error changes to prevent overshoot and simplify tuning. In FRC, potentially useful for velocity control.

Detailed explanation + plots
https://wiki.purduesigbots.com/software/control-algorithms/take-back-half-tbh-controller

Reference implementation
https://github.com/team751/2013RobotCode/blob/master/src/org/team751/speedcontrol/TakeBackHalfSpeedController.java#L98


## State Space control + LQR
This is a more mathematically complex differential-equation based control system, but with many advantages in real-world applications.
https://docs.wpilib.org/en/stable/docs/software/advanced-controls/state-space/state-space-intro.html

These can model physical systems *extremely* accurately, and actually are what is used internally when using [[Physics Simulation|Simulation]] .


# ADRC 


https://en.wikipedia.org/wiki/Active_disturbance_rejection_control