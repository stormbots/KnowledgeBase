
## Goals: 
- Understand how swerve works
- Teleop Interactions for existing swerve
- Reading odometry
- Reset/Initialize odometry 

## Success Criteria
- Use an existing Swerve configuration
- Add human inputs for teleop control
- Add basic PID input to a nearby position
- Point the bot at a specific heading
- Point the bot at a specific target

# Roadmap: 
	## Understanding the tools
	
	The Deploy folder
	
	YAGSL for drive base
	
	Odometry for basic location awareness and coordinate axes for Swerve
	
	[SwerveInputStream](https://broncbotz.org/YAGSL-Lib/docs/swervelib/SwerveInputStream.html) for compressed, streamlined control and management
	
	#TODO Managing/muxing multiple inputs into a single drivetrain output
	
	
	b-line for pathing?
	.withTRatioBasedTranslationHandoffs(true)
	
	 PID To Pose?


# The Swerve Framework, YAGSL

YAGSL is a coding framework for handling a lot of swerve setup, management, and includes useful utilities for reducing code involved with 
- Configuring a swerve drive
- Managing multiple configurations for multiple bots
- Streamlining control+inputs and managing motors
- Handling user input in useful ways
- Underlying optimizations that are hard to manage in a custom codebase
among other things. 

# The Yagsl Docs

Yagsl has extensive documentation and troubleshooting guides here
https://yagsl.yassrobotics.com/get-started/tutorial

However, note that these docs are written in an "ai assisted" way and will contain very confusing details, such as using old apis, unreleased apis, or simply incorrect function calls or values. 

If something doesn't make sense, check with a mentor.

# The Config Files


> [!WARNiNG] 2027  Update Incoming
> The Biocore season will have significant changes to swerve setup details, such as method names, json values, and meaning behind some values.
> 
> This section represents a 2026 season code sample, and shows the main concepts of creating/managing swerves. That will remain the same.
> 


YAGSL does not track all your constants in code like we typically do for other bot-related values. Instead, it uses some external configuration in config files. These are located in a special Deploy folder, which is pushed to the robot alongside the code. 
# Tuning and setting up a new Swerve Drive 
Bringing up a brand new serve drive is a very detail-oriented process with a lot of steps. We won't cover this here, but the Yagsl docs do a good job here. It's recommended to follow their guide when possible:

https://yagsl.yassrobotics.com/get-started/tutorial

When following this guide, it's recommended to do so on a new project dedicated for this tuning.  This allows you to copy+paste the code templates during the setup that will include telemetry data specific to this process. These templates are great for this process, but structured poorly for a codebase thats going to competition!

Once you've completed that process, the resulting config files and the import loading code can be easily brought into your other codebases.

### The Deploy folder

This is a special folder that gets copied to the robot alongside your new code every deploy. This allows non-java files to be kept up to date, and stored on the robot when you make changes.

Yagsl uses this to hold the config parameters, which are loaded at robot boot. You can see this on the File manager in VSCode, and once set up, will resemble something like this:

```
- src/main
  - deploy/
    - (robotname)/
	  - controllerproperties.json
	  - swervedrive.json
	  - modules/
        - (specific module name).json
		- (other modules...).json
		- physicalproperties.json
		- pidfproperties.json
  - java/frc/robot/
    - (your normal robot code goes here)
```

Because the configs are separated by the name, you  can have multiple configs per project, allowing the same code to run on multiple bots without a lot of code changes. 

Also, because these configs exist on the robot, that means you can use them _even if_ you don't have them in your current project! This can be very useful for quick bringups. 

It's often a good  plan to leave a "gold standard', well tuned config on the robot for people who just want to work on a specific bot detail that requires a swerve drive (such as vision, testing, autos, or driver input testing), and use a different name when doing work like learning swerve tuning+creation so you don't overwrite those tested configurations.

In many cases, creating the config only needs to be done once per robot, and that configuration can be preserved as a useful team asset. 


## Structuring your Swerve project

The swerve system gets asked to do a *lot* of different jobs, and setting it up to do all those jobs can be tricky.  There will be a full, commented code template later, but this part will go over some of the basics, and why it might differ from subsystems you've built so far.

Here's the initial code, and then we'll walk through some of the new parts and explain things.

```java
// Inside SwerveDriveSubsystem.java
public class SwerveDriveSubsystem extends SubsystemBase{
  public final SwerveDrive drive;

  /** We'll discuss this in detail below */
  private final SwerveInputStream defaultInputStream;

  /**
   * Unlike most systems, our swerve needs the joystick inputs
   * we plan to use.
   */
  public SwerveDriveSubsystem(
    DoubleSupplier xThrottle,
    DoubleSupplier yThrottle, 
    DoubleSupplier rotationThrottle
  ){
    //Query the bot's Preferences table to figure out 
    // which swerve configuration we're using
    var botname = Preferences.getString("BotName", "compbot");
	
	// Try/catch is how java handles operatiosn that can fail. 
	// This can be because we can't find the config files, or
	// If the config files didn't process right.
    try {
      //Now, load that configuration.
      File swerveJsonDirectory = new File(
        Filesystem.getDeployDirectory(),
        botname
      );
    
    // and build our actual swerve drive
      drive = new SwerveParser(swerveJsonDirectory)
          .createSwerveDrive(5/*mps*/, new Pose2d(6,4,new Rotation2d));
    } catch (Exception e) {
      // In this case we cannot drive the bot.
      // Throwing this error will halt robot code, 
      // and someone must fix it before it can run,
      // but we want to provide some diagnostic info
      System.err.println("Could not find robot config for " + botname);
      throw new RuntimeException(e);
    }

    //Set up our default input stream with basic driver inputs.
    // Your exact settings may differ based on bot and driver preferences. 
    defaultInputStream = SwerveInputStream.of(
      drive, 
      xThrottle,
      yThrottle
    )
    .withControllerRotationAxis(rotationThrottle)
    .deadband(0.05)
    .scaleTranslation(0.6)
    .scaleRotation(0.4)
    .allianceRelativeControl(true)
    ;
    
    //Register our default drive command
    setDefaultCommand(driveFieldRelative(defaultInputStream));

    //Log Yagsl's default field to Glass/Elastic to visualize our position
    SmartDashboard.putData("Swerve/swervefield",drive.field);
  }

  /**
   * Run the bot with a field-oriented notation
   * @param stream The input stream we've built and are using for the bot
   * @return the command to run
   */
  public Command driveFieldRelative(SwerveInputStream stream){
    return run(()->{
	    drive.driveFieldOriented(stream.get(), Translation2d.kZero);
	})
    .withName("Drive Field Relative");
  }

  /**
   * Zero the gyro angle ; Drivers may need this to re-align the bot during matches, 
   * so it's worth putting on their controller somewhere.
   * @return
   */
  public Command zeroGyro(){
    return Commands.runOnce(drive::zeroGyro, this).withName("Zero Gyro");
  }

```


and in RobotContainer.java

```java 
public class RobotContainer {	
  //Our controller, if we don't have one
  private final CommandXboxController joystick = new CommandXboxController(0);

  // Our joystick inputs get passed directly to this. 
  private final SwerveDriveSubsystem swervedrive = new SwerveDriveSubsystem(
    joystick::getLeftY,
    joystick::getLeftX,
    joystick::getRightX
  );

  // ... Rest of robotcontainer
}
```

And now we can discuss some of the surprises
### The Constructor: Creating the Swerve Object

### The Bot's Name

This line is responsible for managing the bot name
```java
    var botname = Preferences.getString("BotName", "compbot");
```

`Preferences` is just like `SmartDashboard`, but persists through reboots. Whatever name is set under the key (`Botname`) is saved, and will be loaded by this file, even if our code hasn't set it. 
If that value doesn't exist, then we use the default, in this case `compbot`.

This lets us give a robot a specific name to map to that particular bot's swerve configuration  (such as Tabi, Mjoulnir, Skipper, or CompBot). This makes it very easy to create a Swerve config that's easily loaded from projects that 

In this case, we want to load whatever config that bot is named, and if we can't detect a name, we'll use `"compbot"`. In practice, we want to default to the name our comp bot uses; This way, if we do a sudden hardware swap in a competition, but didn't have time to set the right name, we'll load the intended config.

For our personal projects, we might want to just ignore this entirely, and do a more simple name assignment. 
```java
    //var botname = Preferences.getString("BotName", "compbot");
    var botname = "myBotName";
```
### Try/Catch

Unlike most of our systems, the use of the "loadable configs" actually generates a "runtime" failure mode. Meaning, it's not an issue in the compiled code; It's something outside your project that the robot is interacting with. In this case, those config files. 

Instead of your usual red squiggles when something goes wrong, in this case, the whole robot fails to boot, and you get "No Robot Code" on the driver station.

While Java (and other code languages) offer ways to work around it, we don't take advantage of this. Because the swerve is so key to playing the game, there's functionally nothing we can do if it doesn't load. So, we simply see the issue, emit a notice, and let the robot stop running. 

This is represented by java's `try/catch` syntax

```java

try{
  // Some operation that can fail 
}
catch(exception...){
 // what do to when it fails.
}

```

The main reasons your code will fail will be 
- Having a typo/error in your config name
- Trying to load a config that does not exist on the robot
- Your config is not correct/complete
- You're loading a 2026 config in a 2027 robot
- You're loading a 2027 config in a 2026 robot
### Loading the config file + Creating a drive from it

> [!WARNiNG] 2027  Update Incoming
> The Biocore season changes the code process used to load these configs.
> 
> This section represents a 2026 season code sample, but the general routine is similar.
> 
> 

Inside our `try` block, we handle the fallable operations: Loading the config itself, and then processing it into a SwerveDrive object.

This is just as required by Yagsl . Some bots may need a couple additional configs (for certain gyro setups), and in 2027 there will likely be differences. 

The only surprise is that you require a start position to create the robot. On a full bot, the chosen location is irrelevant ; A vision system or auto will re-assert your starting position. However, for now we have chosen roughly the center of the field,which will help with some debugging and initial usage.

### The Field2d object

The Field2d object provides a visual reference for the Robot's position on the field. Yagsl creates this by default with useful diagnostic information, and we just need to put it on the SmartDashboard to use it. 

### The Zero Gyro command

The final standalone detail is the zeroGyro command.  This allows the driver to set the gyro to a known heading, which lets them drive accurately.
```java
  public Command zeroGyro(){
    return Commands.runOnce(drive::zeroGyro, this).withName("Zero Gyro");
  }
```

In normal operation, this command is not needed; A vision/odometry system will set the position and robot orientation. 

This command is essential in case a vision system is not working, or not present, such as 
- If it's damaged or disconnected during a match
- If there's no vision system
- If the current room doesn't have vision targets, such as during practice

This means during practice and early season, this is a standard action when testing the bot. Driving a swerve bot without reliable control is a safety hazard.

In competition, it's a last resort in case your vision/odometry system gets damaged, and will allow drivers to continue playing the match.

As such, it's worth ensuring it works and is correctly bound to a button (usually Start or Back).

## Controlling the Swerve Drive SwerveInputStream

The rest of the code we'll discuss revolves around SwerveInputStream. This is a utility class that contains helper functions for converting driver intent into swerve drive actions.  

This class has a *lot* of options! At any given time, only a few options are relevant, and our code is based around this concept. This allows for more clearly defined splits, and prevents some issues that happen when you try to create some large, unweildly input streams. 

## The DriveFieldRelative Command

Most of what we'll work on next is based on this simple looking Command, which hides a few neat details:

```java
  public Command driveFieldRelative(SwerveInputStream stream){
    return run(()->{
	    drive.driveFieldOriented(stream.get(), Translation2d.kZero);
	})
    .withName("Drive Field Relative");
  }
```

`drive.driveFieldOriented` means you're controlling based on the field and gyro, _not_ on what direction the robot happens to be facing. So, "up" on the joystick is always means "away from the driver station", not robot oriented as in "the direction the robot considers the front" like you see on a Tank Drive style bot.

You *can* drive RobotOriented  if you want, which has value in some special cases, such as 
- When drivers are using cameras to see "from the robot" perspective when they cannot see the robot itself
- When doing a vision or sensor lineup where you may have relative data, but not Poses or field coordinates
- Some particular object/field interactions may benefit from it
- Testing the Swerve drive and modules

However, in general, Swerve bots are very difficult for drivers to control from "Robot oriented" notation, so it's advised to use it it as a special case, not as your usual operation. You can also mix-and-match using `drive.DriveFieldAndRobotRelative(...)` to benefit from special cases without interrupting the driver's handling.


Also note the Translation2d being provided; This represents the X+Y coordinates being used as the robot's center of rotation, in "robot oriented" space, with +X being the front of the robot. 
Moving the Center of rotation can allow the driver's "rotate" command to pivot around a specified location, which might make certain robot->field interactions or auto sequences easier. 

![[SwerveCenterOfRotationShift.svg]]


### Default Swerve Input Stream

Let's loop back to our constructor, and review these parts: 
```java
public class SwerveDriveSubsystem extends SubsystemBase{
  public final SwerveDrive drive;

  /** We'll discuss this in detail below */
  private final SwerveInputStream defaultInputStream;

  public SwerveDriveSubsystem(
    DoubleSupplier xThrottle,
    DoubleSupplier yThrottle, 
    DoubleSupplier rotationThrottle
  ){
	// ... Initialization stuff 
	
    defaultInputStream = SwerveInputStream.of(
      drive, 
      xThrottle,
      yThrottle
    )
    .withControllerRotationAxis(rotationThrottle)
    .deadband(0.05)
    .scaleTranslation(0.6)
    .scaleRotation(0.4)
    .allianceRelativeControl(true)
    ;
    
    //Register our default drive command
    setDefaultCommand(driveFieldRelative(defaultInputStream));

```


We see that we pass in our joystick axes in as as DoubleSuppliers . In most subsystems we avoid this, as we can just use Commands to specify action we want from that system. 

However, in our case, *many* swerve actions will need the basic input from the human to travel the field, and we want to avoid typing it out a lot, missing settings that impact driver handling, or otherwise creating bugs.

We also, of course, need these to generate our default "just drive normally" command!

This particular "default" does the following:
```java
	// This creates the initial entry point with the translation throttle values
    defaultInputStream = SwerveInputStream.of(
      drive, //This is our yagsl swerve object
      xThrottle,
      yThrottle
    )
    .withControllerRotationAxis(rotationThrottle)
    
    //This lets us ignore small joystick values. 
    // We normally want this at/near zero, but values near 0.05-0.10 can prevent 
    // old, driving joysticks from moving a robot accidentally
    .deadband(0.05)
    
    //These "scale" the inputs, making the bot go slower
    // You normally want these at 1 for a real bot. However, 
	// our bots and driving area are very small, so this helps 
	// keep them moving slowly and safely.
    .scaleTranslation(0.6)
    .scaleRotation(0.4)
    //Adjust our field-relative inputs based on which side of the field 
    // our drivers are on. So  "up" is away, and "right" is right.
    .allianceRelativeControl(true)
    ;
```


## RobotContainer

Lastly, we just need to set this up so we can run it

```java
public class RobotContainer {
  private final CommandXboxController joystick = new CommandXboxController(0);

  /* These all need to be inverted due to how joysticks present values, and
   * how our drive system expects things to be
  */
  private final SwerveDriveSubsystem swervedrive = new SwerveDriveSubsystem(
    ()-> -joystick.getLeftY(),
    ()-> -joystick.getLeftX(),
    ()-> -joystick.getRightX()
  );


  public RobotContainer(){
    configureBindings();
  }

  private void configureBindings(){
    //This resets the gyro and can be used to help get the field-oriented
    // drive to have forward/back/left/right feel correct
    joystick.back().onTrue(swervedrive.zeroGyro());
  }



```




## Fancier Actions

Next, you may want to check out [[Swerve Simulation]] ; This  helps you test your swerve code without needing the hardware. 

There's a lot you can do using SwerveInputStream, which is the topic of [[Swerve Motion]], which covers point-to-point movement, aiming, and creating custom Swerve actions.

Many good swerve interactions also rely on [[Swerve Odometry]] , which also relies on 