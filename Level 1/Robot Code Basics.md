[](Basic%20Motor%20Control.md)[](Basic%20Motor%20Control.md)[[Motor Control]]
[[Commands]]
[[Encoder Basics]]


## Success Criteria
- [ ] Install the WPILib VS Code IDE
- [ ] Make a new robot project
- [ ] Create a new me
- [ ] Install the Rev third Library
- [ ] Basic requirements to start working  on robot projects
- [ ] Create a new empty subsystem 
- [ ] Create a new empty command
- [ ] Add your new command and subsystem to RobotContainer.

#### Goals 

Programming prerequisites, listed in [[Coding Basics]] for now


https://docs.wpilib.org/en/stable/docs/software/commandbased/structuring-command-based-project.html

# Robot Code structure

When you open a new robot project you'll see a lot of files we'll interact with.

- `src/main`
	- `deploy`
	- `java`
	- `frc/frc/robot`
		- `commands`
			- `ExampleCommand.java`
		- `subsystems`
			- `ExampleSubsystem.java`
		- `Constants.java`
		- `Main.java`
		- `Robot.java`
		- `RobotContainer.java`
- `vendordeps`

For typical projects, you'll be spending the most time in `RobotContainer`, `subsystems`, and occasionally `commands`. 

For some early practice projects, or special use cases you might also interact with `Robot.java`,  `ExampleCommand.java`, or `ExampleSubsystem.java` a bit.

## Third Party Libraries

Many helpful utilities we'll use for robot projects are represented using code that's not available by default. WPILib has a small manager to assist with installing these, detailed here: 

https://docs.wpilib.org/en/latest/docs/software/vscode-overview/3rd-party-libraries.html#adding-offline-libraries

## Third Party Tools
We'll also utilize a number of software tools for special interactions with hardware or software components. Some of these include 
- [[Rev Hardware Client]] , a motor controller management tool
- [[LaserCan|GrappleHook]] , which manages the LaserCan configuration
- [NI Driver Station](https://docs.wpilib.org/en/stable/docs/software/driverstation/driver-station.html) is needed for systems that will want to enable and drive a robot. 

## Putting Things in the Proper Place

The hardest part of getting started with robots is figuring out _where_ your robot code goes. 

#### Robot.java : A microcosm of the complete robot

`Robot.java` is a very powerful file, and it's possible to write your _entire_ robot in just this one file! For reasons we'll get into later, we _do not_ want to do this. However, the setup of it does a good job explaining how a robot works. Let's look at the structure of this file for now

```java
public class Robot extends TimedRobot {
	private Command m_autonomousCommand;
	private final RobotContainer m_robotContainer;
	public Robot() {
		m_robotContainer = new RobotContainer();
	}
	
	public void robotPeriodic() {}
	
	public void disabledInit() {}
	public void disabledPeriodic() {}
		
	public void autonomousInit() {}
	public void autonomousPeriodic() {}
	
	public void teleopInit() {}
	public void teleopPeriodic() {}
	
	public void testInit() {}
	public void testPeriodic() {}
	
	//a few more ignored bits for now
}
```


From the pairing, we can group these into several different modes
- "Robot"
- Autonomous
- Teleop
- Test

Indeed, if we look at our [[Driver Station]], we see several modes mentioned.
![[driverstation.jpg]]
Teleop, Auto, and Test are simply selectable operational modes. However, you might want to utilize each one slightly differently.

"Practice mode" is intended to simulate real matches: This DriverStation mode runs Autonomous mode for 15 seconds, and then Teleop Mode for the remainder of a match time. 

"Disabled" mode is automatically selected whenever the robot is not enabled. This includes when the robot boots up, as well as as whenever you hit "disabled" on the driver station.
Disabled mode will _also_ cancel any running [[Commands]] . 

#todo your mentor did something dumb while writing this, paragraph 
> "Robot mode" isn't an explicit mode: Instead, of "Robot Init", we just use the constructor: It runs when the robot boots up. In most cases, the primary task of this is to set up Robot Container.
`robotPeriodic` just runs every loop, regardless of what other loop is also running.


We can also see a grouping of 
- Init
- Periodic
Whenever any new "mode" starts, we first run the Init function once, and _then_ we run the periodic. The robot will continue run associated Periodic functions every loop, 50 times per second. 

We generally won't add much code in Robot.java, but understanding how it works is a helpful starting point to understanding the robot itself. 

#### RobotContainer.java
As mentioned above, the "Robot Container" is created when you create a new project, and one of the first things that gets executed when running robot code. 

This file contains a small number of functions and examples to help you organized. 

```java
public class RobotContainer{
	ExampleSubsystem subsystem = new ExampleSubsystem();
	ExampleCommand command = new ExampleCommand();
	CommandXboxJoystick joystick = new CommandXboxJoystick(0);
	RobotContainer(){
		configureBindings();
	}
	public void configureBindings(){
		//Not a special function; Just intended to help organize 
	}
	public Command getAutonomousCommand(){/*stuff*/}
}
```

This file introduces a couple new concepts  
- [[Commands]], which form the  "actions" you want the robot to perform
- [[Subsystems]], or the different parts of the robot that could perform actions
- [[Joysticks]] , which serve as the standard input method.

The use of Commands and Subsystems goes a *long* way to managing complex robot interactions across many subsystems. However, they're certainly tricky concepts to get right off the bat. 

#### Constants.java
Sometimes, you'll have oddball constants that you need to access in multiple places in your code. Constants.java advertises itself as a place to sort and organize these constants. 

Without getting too into the "why", in general you should minimize use of Constants.java; It often leads to several problems as your robot complexity increases. 

Instead, keep your constants as close to where they're used as possible, and move them up through the robot hierarchy as necessary. This is known as  "scope management" or "encapsulation".

- If a value is used once, just use the value directly. This covers a lot of setup values like PID tuning values.
- If your value is used repeatedly inside a subsystem, make it a private constant in that subsystem. This is common for conversion factors, min/max values, or paired output values
- If a constant is strongly associated with a subsystem, but needs to be referenced elsewhere, make it a public constant in that subsystem.
- Lastly, if something is not associated with a subsystem, never changes, and used repeatedly across multiple subsystems, Constants.java is the place. 

Remember, you can change your mind later! If you find yourself depending on a lot of constants, you might need to consider [[Refactoring]] your code a bit to streamline things. Note that Stormbots code generally has almost nothing in here! 


#### ExampleSubsystem.java

The ExampleSubsystem.java file is automatically created when creating new projects, giving you a blank [[Subsystems|Subsystem]]. We'll explore Subsystems in depth later.

By default, this class is created, and the `periodic()` function it contains will run every code loop. This makes it a great place to put a simple, example code with minimal setup and fuss. 

#### ExampleCommand.java

This is a standard, blank [[Commands|Command]] . By itself, this file is not too useful. However, the fact that it exists allows example joystick interactions to be set up in `RobotContainer.java`.


## Understanding The Examples

The code examples provided by the default templates (ExampleSubsystem and ExampleCommand) may or may not be used in our examples.

For our purposes, they should work as expected without a deeper understanding of Command based robots while you get your footing, and enabling you to migrate to Command based robots once you get there.

In general, our examples will indicate where to put things, and point you the right way.  If an example doesn't talk about Command based stuff, you probably don't have to worry about it!


## Creating new files

While you *can* just create new files in a "normal" way, it's easy to accidentally miss code details that make things harder for yourself. 

Instead, WPILib+VSCode has a built in way to do this. Simply right click the appropriate folder, and find the "Create new class/command" option.

![[creating-new-command-1.png]]

This will present you with a list of common FRC classes. The typical ones you'd use are Command and Subsystem.

![[creating-new-command-2.png]]

## Code Flow

For those familiar with coding, you might wonder "Where does code actually start from?". This is a fair question in FRC code, which is a more complicated framework. The code path itself is predictable, if not a bit obscured. A simplified view looks like this

```
- Main Bot Initialization
  - Robot.java::RobotInit
    - RobotContainer
      - Subsystem Constructors
      - Command constructors
  - Enter Disabled mode
    
- Disabled Mode
  - Initialize 
    - Robot.java::DisabledInit()
  - Loop
    - Robot.java::DisabledPeriodic
    - (each subsystem)::periodic()
      
- Teleop Mode
  - Initialize 
    - Robot.java::TeleopInit()
    - (any active commands)::init()
  - Loop
    - Robot.java::TeleopPeriodic()
    - (each subsystem)::periodic()
    - (any active commands)::execute()
```

This is a *lot* of complexity if you're new to coding, but don't panic. You can *mostly* ignore a lot of this complexity, and our examples will help you position things properly until you get your bearings. 

The complex part, robot actions that move motors and do things, basically boil down into
```
initialize/prep on boot
disabled
initialize
run
stop
```
We will quickly capture this with our [[Commands|Commands]] framework, enabling you to do simple things very quickly and efficiently, like they're self-contained little programs.


## Deploying Code

Once your code is written, then it's time to [[Deploying Code|Deploy Code]]

