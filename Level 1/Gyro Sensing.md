---
tags:
  - stub
  - hardware
  - sensor
aliases:
  - IMU
  - Gyro
---




## Success Criteria
- [ ] Configure a NavX or gyro on the robot
- [ ] Find a way to zero the sensor when the robot is enabled in auto (to establish a forward direction)
- [ ] Create a command that tells you when the robot is pointed the same way as when it started
- [ ] Print the difference between the robot's starting angle and current angle




## Other Uses

#### As an Odometry component
IMUs are often used for [[Basic Odometry+Telemetry|Field Odometry]] , tracking the robot position to facilitate precise control. In these cases, we do not read the IMU directly, but simply let the odometry tools handle it.

#### As a "Tilt" Sensor
Gyros can be used directly to identify the tilt of a robot; This is very useful in games where an objective is to "balance" the robot against a field element.
#### As an angular sensor
While uncommon on our team, you can use Gyros directly on mechanisms like [[SuperStructure Arm|Arms]] or [[Superstructure Shooter|Shooters]] to identify their angle