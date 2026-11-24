
## Goals: 
- Create drive commands to drive to a target point
- Create drive commands to aim the bot at targets
- Create simple sequences using other Swerve commands


> [!TIP] Odometry is useful here!
> Many advanced commands need the swerve odometry to work correctly.
> 
> While you can learn and test these bits of code, you may get unexpected results, or have difficulty validating them on hardware.
> 
> Note that using [[Swerve Simulation]] may help with this, as that provides some basic odometry and a safe environment for you.

## Remixing our inputs and creating commands

Most swerve bots will want at least one alternate drive command that sets the `scaleTranslation/ScaleRotation` values as a "slow mode" when drivers are trying to do something precise! Or, sometimes we want to default as slow, and make a "sprint" button that unlocks full robot speed.

So, let's make one.

First, our standard structure for a new Input command will look like this:

```java
  public Command nameOfTask(/*Any useful parameter data*/){
    var inputs = defaultInputStream
    .copy()
	/* Additional tweaks to our  input stream*/
    ;
    return driveFieldRelative(inputs);
  }
```

The `.copy()` creates a new, modifiable instance of our input stream unique to this command. We can then safely modify it without impacting our defaults or other commands.

Our entire job is now reduced to creating the adjusted input stream, an then passing it to the command we already made! Easy! 

In this case, we're changing our default from slow to less slow, so let's name it Sprint. We have no parameters, so let's remove those.

```java
  public Command sprint(){
    var inputs = defaultInputStream
    .copy()
	/* Additional tweaks to our  input stream*/
	.scaleTranslation(1)
    .scaleRotation(1)
    ;
    return driveFieldRelative(inputs);
  }
```

That's it! We can now bind this to a button, and we can have two different input speeds on our bot. 

## Aiming at A Target

A common benefit of a swerve is aiming at something other than what you're driving at. A command that handles this is created in much the same way.

```java
  public Command aimAtTarget(Pose2d target){
    var inputs = defaultInputStream
    .copy()
    .aim(target)
    .aimWhile(true)
    ;
    
    return driveFieldRelative(inputs);
  }
```

Since this default still uses the joystick inputs, you still have input without having to fuss with it!

Some tasks like this change the calculation modes of a SwerveInputstream, and need to be "switched on" using a `___While(true)` modifier. These switch between different ways of generating values. In this case, it changes  from reading the joystick axis to control rotation, into a calculated difference between where you are, where you're facing, and where the target is.  

`[!!book-alert|Note:|var(--color-cyan-rgb)]` there are no PID constants in this particular case; There's some internal tricks using the individual wheel module's PID+FF  values that allow it to change the rotation without having to provide them directly.

## Driving To A Position

This one is slightly more complicated, and has some additional surprises. However, at the core it's the same process: 

```java
  private Command driveToPosition(Pose2d goalPose){
    var inputs = defaultInputStream
    .copy()
    //Enable the correct mode; This one completely takes driver control away
    .driveToPoseEnabled(true)
    //This one takes several parameters
    .driveToPose( 
      ()->goalPose, 
      getTranslationPidController(),
      getRotationPIDController()
    )
    //You likely want scaling at 1 (max capability according to ProfiledPid)
    .scaleTranslation(1)
    .scaleRotation(1)
    ;
    
    return driveFieldRelative(inputs);
  }

  private ProfiledPIDController getTranslationPidController(){
    // This is in m/s and m/s/s
    var constraints = new Constraints(3, 5);
    //Our PID inputs are in meters, although output units are unclear 
    var pid = new ProfiledPIDController(2, 0, 0, constraints);
    return pid;
  }

  private ProfiledPIDController getRotationPIDController(){
    // This takes rad/s and rad/s/s , so we use a conversion from degrees
    var constraints = new Constraints(
      Units.degreesToRadians(120),
      Units.degreesToRadians(240)
    );
	//Our PID inputs are in radians, although output units are unclear 
    var pid = new ProfiledPIDController(2.5, 0, 0, constraints);
    return pid;
  }

```

While this *could* be done inside the singular command, it's a bit cleaner to break the PID generators out.

Be sure to watch your units! This is a case where we get very mixed units, as many parameters are in Meters and Radians internally. Using the Units library for conversions, and being consistent and labeling things will pay off. 


> [!TIP] Quick explainer on ProfiledPID
>  If you haven't been exposed to a ProfiledPID, you can treat it similarly to a raw PID controller. However, it's a bit smarter, and does three main things: 
> - Reduces the accelleration when you enable it/change setpoints, so it starts smoothly
> - Constrains the speed to a maximum, defined value
> - Slows the bot down before it gets to the goal position, preventing overshoot.
> 
> As a result, it provides *much* better motion, is simpler to tune, works well with higher gain values, and provides more control. You typically want to set the velocity constraint at the max speed of your robot (~4-6mps). Acceleration you can just start slow and increase until your robot misbehaves, or until it can't keep up. 
> 
> For comparison, you can think of a normal PID controller as having infinite acceleration and infinite velocity.

### Sequencing Motions + Detecting goal conditions

Normally, sequencing motions requires the command to know that it's done, and exiting on it's own. The most useful one to sequence is our driveToPosition command. Moving to two points is very useful, and you could write whole, simple autos with it.

First, we need to figure out how to figure out if we're at a particular position. This is straightforward (you've done this math!), but uses some methods we may not have interacted with.

From your geometry class, you know that the objective is to calculate
```  
    dx = target.x-bot.x
    dy = target.y-bot.y
    dist = sqrt(dx**2+dy**2)
    dist <= tolerance 
```

In code, we can do that like so:
```java
  public boolean isAtPosition(Pose2d target, Distance tolerance){
	// Get a Translation2d, representing X/Y position without the angle
	var position = drive.getPose().getTranslation();
	//We can now use a method to get the distance between those points
	// This handles the dx,dy, and sqrt
	var dist = position.getDistance(target.getTranslation())
	
	//And now we can compare it. Since the internal Pose units 
	// are meters, we have to convert our tolerance distance.
    return dist <= tolerance.in(Meters);
  }
```

Then, just create a variant of our `driveToPosition` with a decorator that exits.
```java
  public Command driveToPosition(Pose2d goalPose, Distance tolerance){
    return driveToPosition(goalPose)
    .until( ()->isAtPosition(goalPose, tolerance) );
  }
```

This code isn't perfect; Try sequencing a small, square motion of 4 points, with a large tolerance (like 24in). The intended goal path isn't quite what will happen, and what you'll see is behavior like this: 

![[SwerveToleranceVsPathing.svg]]

Note you never got to any of your goal positions, including at the end! You'll also see it stop abruptly at each path transition, when you'd expect a smooth start and smooth stop.

What's happening is that the tolerance is cutting the motion short, before your profile can slow down and  get to the goal position. 

Whether this is a *problem* or not depends  on the motion and the game. You might bump into objects, end the sequence misaligned, or not get close enough to successfully place game pieces. 
On the other hand, if you're *too* precise a problem along  the way might cause you to be *just* outside the tolerance, halting your sequence. 

What we can do to fix both is also account for the completion of a motion by simply checking if the robot is where we need it *and* stopped. We need to write a new method for this. The math here doesn't need much explanation, just some conversions

```java
  public boolean isRobotStopped(){
    var speeds = drive.getRobotVelocity();
    //These values are fairly arbitrary; You'd want to adjust them 
    // to what your got+game needs!
    //In some cases, you may want to parameterize them as part of 
    //your "isRobotStopped(...) function
    var stoppedMPS = InchesPerSecond.of(3).in(MetersPerSecond);
    var stoppedRPS = DegreesPerSecond.of(10).in(RadiansPerSecond);

	var speedxy = Math.hypot(
		speeds.vxMetersPerSecond,
		speeds.vyMetersPerSecond
	);

    return speedxy < stoppedMPS
	&& Math.abs(speeds.omegaRadiansPerSecond) < stoppedRPS ;
  }
```

and then add this condition to our new drive command.
```java
  public Command driveToPosition(Pose2d goalPose, Distance tolerance){
    return driveToPosition(goalPose)
    .until( ()->isAtPosition(goalPose, tolerance) && isRobotStopped());
  }

```

Now the robot will *very*  closely track your goal positions, even with a large tolerance. If unobstructed, it'll finish the motion, and won't be stopped until very close to the goal position, and will then exit, and continue the sequence.

This does a great job handling the most common obstruction: bumping into a wall near the goal. That will usually leave you stopped and within tolerance, and exit as intended. If you're too far away and had something like a robot collision, it'll continue the approach.

Other mid-movement obstructions will simply leave the bot trying to drive to the goal position, or the sequence will stall, but in the intended way. EG, if you didn't drive to your destination position, you don't want to try to score your game piece.


## Other things! 

You've probably noticed that there's a *lot* of flexibility here! There's capabilities we haven't touched on, and combinations beyond these basics. 

Depending on the needs of your game/project, you can mix-and match several capabilities at once, to do things like: 

- Face a particular direction (Up/Down/Left/Right) while driving , rather than at a specific target
- Aim at a target while  while auto-driving to a field position
- Aim at a target using the back or side of a robot


