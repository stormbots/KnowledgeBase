## Success Criteria
- [ ] Set up a mock project  with a nominal, standard code structure


Project should have a subsystem that
- Is in a subsystem folder
- Has 3 components in a (logic, [[Physics Simulation]], [[Mechanism2d]])
- Has a factory method to get a control command (can be mocked up)
- Has a trigger that indicates a mechanism state (can be mocked up based on timers)
Has an additional sensor subsystem system that
- Provides a trigger for a condition (can be mocked up)
Has a controller and 
Has an Autos class to hold autos
- With an auto chooser initialization 
- A single basic auto using subsystem and sensor


## Planning your bot structure:
####  High level Superstructure states
Start by thinking about the broader, high level superstructure states. EG, what your bot is doing and what simpler mechanisms are involved with that. 
- Intaking game pieces
- Scoring game pieces
- Climbing/Performing end game

Depending on the construction you might be able to fully decouple many of these states, and even enact multiple at once.

In many systems, however, this is *not* true, with heavy coupling between critical subsystems. This is almost always the case for [[Superstructure Indexer|Passthrough]] style subsystems, which are heavily controlled on both intake and exhaust sides, complicating code.

### Low Level Mechanism States
Next, think about the low level mechanisms: These should be very simplified, single actuator representations
- Intake Rollers
- Shooter Flywheels
- Arm Pivot

While it's tempting to represent these as "groups", avoid the temptation and model individual motors/actuators separately if they're not rigidly linked together (as in a gearbox or shaft)

These should have *very* simple and consistent interfaces: You just want to set the actuator state, position, or get information from it. Broadly, the interfaces should be simplified, such as 
- setPosition/setAngle/setHeight/setVelocity 
- setVoltage (for direct output control)
- getPosition
- getVelocity
- stop

### Mid-Level Grouping

Now is the time to start structuring things to get from the top level generic tasks, to the lower level actuator side.

For this, look for the smallest cluster of mechanisms that independently do a useful task. 

[[SuperStructure Intake|Intakes]] are a good example: Often you need to extend/deploy an intake along with some rollers. Since intakes are often wide, sometimes there's individual actuators on both sides, making this 3 components (left arm, right arm, rollers)

Defining your intake API helps you plan this out: 
- Intake (both arms deployed, and rollers running inward)
- Eject (Rollers running backwards, arms might have some consideration)
- Stop/Stow (both arms retracted, rollers stopped)

Notice, as part of this you can simply assert "When doing X, the mechanisms are doing Y". This is ideal, and what we're building toward.

[[Superstructure Shooter|Shooters]] are another useful grouping example, and show the value of this structural style. These have two major mechanisms (a Flywheel and a Feeder to provide it game pieces), with two critical states:
- Shooting (Running at a goal RPM, and feeding)
- Stopping (Idle/Stop speed and not feeding)
However, you generally have some edge cases:
- You don't want to feed game pieces while trying to get up to speed
- You don't want to stop running while the feeder might be passing game pieces
The mid level group gives you a clean place to handle this internal conflict, such that your high level group does not need to consider it. 

### Increasingly high level

You may need to form additional tiers as you go to fully define your goal states. Usually, you want to draw these boundary lines at places where a group of mechanisms might have specialized tasks, conditions, or groupings. 

The goal of this architectural style is to streamline management, and reduce the number of high-level states you have to juggle. By letting smaller clusters manage themselves, you can usually avoid building absurdly complex [[State Machines]] , and leverage many, smaller state machines. 

In general, we tend to build systems using Commands and Command groups; These are small, self contained state machines, and recognizing them as such helps with this planning. 


### Code Example: 
For this example, we'll refer to a simple shooter bot that has an intake and a shooter system.

First, define the high level concepts 
```java
// Somewhere in robotContainer or a superstructure class
Command shoot = Commands.parallel(
	shooter.shoot(),
);
Command intake = Commands.parallel(
	intake.intake();
);
```

According to this code, we're in the ideal zone where we *could* shoot and intake at once; Our two core states don't interact.

But importantly, these high level definitions are not really managing the details of the system, beyond ensuring that each system has what it needs to make decisions.

Now we get to probe the underpinnings of what makes those commands work.

```java
public Shooter{
	private Flywheel flywheel = new Flywheel();
	private Feeder feeder = new Feeder();
	
	public Command shoot(){
		
		var feed = Commands.repeatingSequence(
			feeder.stop().until(flywheel.isAtTargetVelocity),
			feeder.feed().until(flywheel.isAtTargetVelocity.negate())
		);
		
		Commands.parallel(
			flywheel.setVelocity(/*goal rpm*/),
			feed	
		);
	}
}
```

And a similar one for our intake
```java
public Intake{
	private IntakeArm left = new IntakeArm(/*...*/);
	private IntakeArm right = new IntakeArm(/*...*/);
	private Rollers rollers = new Rollers();
	
	public Command intake(){
		return Commands. parallel(
			left.setPosition(0),
			right.setPosition(0),
			rollers.setVelocity(500)
		)
	}
	
	public Command stow(){
		return Commands. parallel(
			left.setPosition(90),
			right.setPosition(90),
			rollers.stop()
		)
	}
	
}
```


The more complex one is the shooter: You can see that in this case, the Feeder actually has a small state machine: It cycles between waiting for the shooter to be up to speed, and then feeding it game pieces. But, from the perspective of the higher level control, you simply don't care: The shoot command takes care of it. 

On the intake side, the high level simply doesn't care about whether there's one arm, two arms, or the like. You just tell it to intake, and it makes that happen. 

This path generates a fairly consistent, streamlined flow, and works well within the idioms of WPILib and FRC in general. 

### Heavily coupled systems

In cases where a system is *extremely* tightly coupled on many different subsystems, or uninterruptible processes/transitions, this approach might fall apart, or leave you with lots of subsystems to declare with a large number of system states.

In such cases, bundle them as best as you can to reduce the overall number of global states, and then look at alternate abstractions that seem to make sense. 

In some cases, instead of using Commands/Subsystems as a rigid mutex for single-owner control, you might be able to provide multiple inputs at once, and let the subsystem figure out what to do with that. 

For example, a Swerve drive might have drivers trying to reposition it, but the Shooting system needs to control rotation to aim. In such cases, providing both to the drivetrain lets it add them together, and do both at once.

For some systems you might be able to provide multiple "behavior requests", and let it prioritize. For example, a indexer/feeder might have a "load new piece" and "Feed to shooter" request, and determine that the "feed to shooter" is the one to handle at that time.