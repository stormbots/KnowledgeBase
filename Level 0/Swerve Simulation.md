# Goals: 
- Configure your driver station + system for simulation
- Drive and test the simulated robot

# Preamble:

Simulation enables us to drive the robot without actually having to claim the physical system. This enables us to do some very nice things like:
- Run it at full speed
- View the robot on a virtual field, to get a better sense of where it is and where it's aiming
- Test our  code at home
- Test our code while someone is validating it on a robot
- Verify some code processes while the build teams are working on the hardware


### Not covered: Other mechanisms

A *lot* of the robot can be simulated in much the same way! However, setting it up is often more involved, particularly when you want helpful visualizations. YAGSL has sim built in, but other  mechanisms don't, or are more challenging to understand from their simulated data.

So, be mindful that if you're running the sim on a full robot, sequences involving those mechanisms may not complete as they would on a real robot. 
# Using the DriverStation app


# Using Glass/Sim interface