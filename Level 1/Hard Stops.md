---
tags:
  - hardware
  - sensor
---



## Synopsis
A hard stop is a  physical "end of travel" for a system. 

This is common on most positional systems (such as [[SuperStructure Arm|Arms]] or [[SuperStructure Elevator|Elevators]]), and generally uncommon on velocity systems ([[SuperStructure Rollers|Rollers]], [[SuperStructure Flywheel|Flywheels]]). 

## Practical uses

Hard stops serve as a physical reference point for [[Encoder Basics|Relative Encoder]] systems, which are present on all motors. These allow our systems to have accurate, repeatable placements through the course of a season, and a key part of operating encoder-driven systems.

Hard stops can be used as a reference directly (starting the robot against the hard stop), or referenced via a [[Homing Sequences|Homing Sequence]] via code.￼￼Success Criteria
￼￼￼￼￼ Configure a NavX or gyro on the robot
￼￼￼￼￼ Find a way to zero the sensor when the robot is enabled in auto
￼￼￼￼￼ Create a command that tells you when the robot is pointed the same way as when it started
￼￼￼￼￼ Print the difference between the robot's starting angle and current angle


- TODO
- what's an mxp
- what port/interface to use, usb
- which axis are you reading
- 

## Hard Stop Best Practices

#### Make them robust + Stable
In order to serve as a reference, a hard stop must remain in place. In order to ensure contact, both humans and code will drive the system into the hard stop, but usually in a controlled manner. 

This should be recognized in the design phase, and ensure that hard stops and contact points can survive repeated interactions.

#### Design systems to avoid damaging hard stops
This is partly mechanical: System should be geared such that when powered off or disabled unexpectedly, the system does not crash into hard stops. The system should either hold position or fall at an acceptable velocity.

This is partly code: During tuning, you can set soft limits away from hard stops to prevent tuning faults from crashing the system into the hard stop. If practical, you can set soft limits away from hard stops in production code as well.

#### Leave them be
Do not move hard stops. Because these are reference points for a physical system, hard stop changes can disrupt the positioning used by critical bot actions.  

Linear systems (Elevators) rely strongly on hard stops for precision: Hard stop changes can result in poses and heights being off, which can impact precision tasks. In general, this is a lesser issue. 

Angular system (Arm+Shooter) are are strongly affected by hard stop changes. When driven by [[Encoder Basics|Relative Encoders]], you significantly impact precision: Every degree of error caused by a hard stop change is magnified by the distance to the end effector or target. Even small changes of a few degrees will caused missed shots or placements.

When pivoting systems are driven by [[Absolute Encoders]], moving a hard stop _into_ the working space of the system will force the system to actively fight the hard stop. Because the acceleration profiles do not account for this intrusion, every motion to "park" will crash into the hard stop at high speeds, damaging to the system.

If the hard stop of a system seems like something that will likely experience damage through the season, consider an intentional hard stop design that can have a consumable plate or bar that can be replaced. 

If a surprise or emergency repair occurs, make sure to involve programmers and validate both soft limits and motion profiles to ensure the new limits do not interfere.