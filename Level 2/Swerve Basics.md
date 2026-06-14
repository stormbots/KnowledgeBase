
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

## Understanding the tools

The Deploy folder

YAGSL for drive base

Odometry for basic location awareness and coordinate axes for Swerve

[SwerveInputStream](https://broncbotz.org/YAGSL-Lib/docs/swervelib/SwerveInputStream.html) for compressed, streamlined control and management

#TODO Managing/muxing multiple inputs into a single drivetrain output


b-line for pathing?
.withTRatioBasedTranslationHandoffs(true)

 PID To Pose?