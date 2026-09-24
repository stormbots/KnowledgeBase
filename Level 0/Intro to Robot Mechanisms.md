
### Goals: 
- Provide vocabulary to discuss mechanical mechanisms
- Provide an understanding of what these mechanisms can do
- Provide tools to identify and analyze the function of these systems in a robot

# Overview
A key aspect of any robot is it's mechanical systems: These define how the robot can move and interact with the world. 

A mechanical system starts with a motor, and depending on it's placement, speed, power, and attachments, define that system's intent, and what it can do. Systems that seem very similar can sever very different roles and have very different ways of making them work properly. 

The mechanisms we will discuss are broadly generic across robots, but we'll be using the aid of some existing robots to provide clear references that you will have access to over the season. 

### Introducing: The 2026 Kitbot.
![[Kitbot 2026.png]]

The 2026 Kitbot, built for the game Rebuilt, has three tasks: 
- Load yellow balls, called fuel
- Store the fuel temporarily
- Shoot the stored fuel into a scoring objective. 

The Kitbot has 4 distinct "mechanisms", which work together to perform these tasks effectively. 

- A Drivetrain
- The Intake
- A selector system, affectionately known as the "doorman"
- The Flywheel

The storage system (called a hopper) is simply a passive bin

# The Drivetrain
You've probably already messed with this part! This is the wheels that the robot sits on, and is used to move the robot around. 

Drive trains are a critical robot aspect, and for their apparent simplicity, take a *lot* of work to get right. 

# The Intake

An "Intake" is responsible for the initial loading process of a game piece.

For  the 2026 kitbot, the Intake is these two bars at the front opening of the robot. 

![[Kitbot 2026 Intake.png]]

Intake systems are special among FRC systems, because they have so many uncertainties. 
- You may not know if (or when) it's interacting with a game piece.
- It may interact with *many* game pieces at once
- They're often exposed, and may interact with walls, field elements, or other robots.
- It may be interacting with abnormal game pieces (torn, damaged, or oblong ones in some games)

Because of these, intakes are built to be especially durable, and usually given a lot of power. A broken intake usually means you are no longer able to play the game in any useful form. That is very undesirable, and frustrating for drivers.

This intake works by simply spinning your motor one way to load balls into the robot, and spinning it the other way to unload them. (this is actually a bit more complicated as we'll see later).
# Roller Systems
More generally, this intake is an example of a "Roller" class of mechanisms. Roller systems are a very simple, basic mechanism. They are usually defined by 
- Being stationary mechanisms
- Having one or more wheels or bars that interface with a controlled element (usually our game piece)
- Causing the translation (movement) of a controlled item (and sometimes rotating it in the process)

Rollers are *very* common on FRC bots. So common in fact, that we rarely call something "Rollers", and instead focus on what special purpose that roller is designed for. All 3 systems on the kitbot, in fact, are roller systems with fancy roles! 

# The Flywheel (aka, the Shooter)

In FRC, a "Shooter" is a system for launching game pieces out of the robot; Usually at high speeds, and usually with a fairly high accuracy and repeatability. 

We generally don't refer to a single part of the robot as a "Shooter"; This is because shooting requires multiple parts of the robot working together to function. So we reserve "Shooter" as a group term, and use more specific terms for other aspects.

For this kitbot, our "Shooter" is mostly is two parts:
- a passive mechanical backplane
- our Flywheel system
- And a couple supporting systems


![[Kitbot 2026 Flywheel.png]]

A Flywheel is a common industry term to describe a spinning mass that stores energy. This is *exactly* what we want when we need to accelerate a game piece quickly, and with precise speed control, so we borrow that name.

A Flywheel is usually the key aspect of any Shooter system. This Shooter system is a "Fixed Angle Shooter". The Backplane controls the exit angle of the fuel, and the Flywheel speed controls the exit speed. Together, you can adjust your shots to adjust for the required distance and height to score. 

# The Doorman

The "Doorman" is a roller system that is sandwiched between the Intake and the Hopper (fuel storage bin), as well as between the Hopper and the Flywheel. It's called the "Doorman" because it's job is to control what goes in, and what goes out.

This means, depending on what you're trying to do, the Doorman acts as part of the intake, or as part of the shooter! 

![[Kitbot 2026 Doorman.png]]

Understanding the function and operation of the Doorman (and many similar systems) involves looking at the robot as a functional whole. On it's own, it does very little, and without it, the other systems don't work either!
### The Doorman During Intaking

When intaking fuel off the floor, the "Doorman" is effectively a third intake roller. It spins the same way, and just helps funnel fuel into the robot. 

![[Kitbot Roller Path Intake.svg]]
### The Doorman during Ejecting

If running forward pulls fuel in, then running backward should remove them. And indeed it does.

![[Kitbot Roller Path Eject.svg]]


### The Doorman during Shooting

You might note that the Shooter/Flywheel sits high above the intake path, and has no contact with fuel. No matter how fast you spin it, this system can't do much. 

For the Kitbot, the Doorman  helps with this. Running the doorman to pull fuel from the hopper, and running the intake inward funnels fuel upward to the shooter. The intake can also assist with this, although it shouldn't be necessary.

![[Kitbot Roller Path Shoot.svg]]

This arrangement is very common with shooters; Flywheels are normally difficult to get spinning fast if they're in contact with game pieces, so you need a secondary mechanism to assist.

There's some hidden complexity here. Flywheels often need some amount of time to spin up to their target speed, and cannot be in contact with game pieces during that time. (try it out to see what happens!). So, you often need some sort of delay between when you start spinning your Flywheel and when the Doorman passes fuel to it. 

Similarly, when shooting multiple game pieces at once, you may find your Flywheel losing energy and start missing shots. You may need to stop the Doorman to interrupt the feeding process to wait until the Flywheel is back up to speed.


So, the Doorman's job is to h. However, note that the Flywheel has to store a *lot* of energy, and get to spinning *very* fast. This means, you usually want the Doorman to wait until the Flywheel is actually at the speed we want. This makes it's role in the shooting process a bit more complicated than the one for intaking. 
# The Arm System

Our Kitbot is decked out with a bonus feature: Two arms that come down to help funnel fuel into our Intake. 

![[Kitbot Arm Path.svg]]

Arms, or Pivot mechanisms are systems that hinge around one end. Sometimes these house additional systems (often Rollers).

Control of pivoting systems is often *much* more difficult than roller systems! Rollers can be run in either direction indefinitely without any problems. 

However, when an Arm runs, the mechanism moves, and will eventually hit something. Continuing to run the motor past that point can cause damage. In addition, Arms will respond differently to a given motor output depending on their position.

Because of this, Arm control can often be more complicated than other systems, and may require clever solutions to have them work reliably. Arms usually are not included on standard Kitbot designs for this reason. 


# Linear Motion Systems

The final often seen mechanism type is one where the motor results in something moving in a straight line. Usually, this means up and down, but sometimes things are extended out and retracted. 

Like Arms, Kitbots do not commonly include Linear mo