
Requires:
[[FeedForwards]]
[[PID]]
Reading Resources: 
[[Homing]]
## Success Criteria
- [ ] Create an Elevator subsystem
- [ ] Set Encoders
	- [ ] find the encoder range and conversion to have real-world values
	- [ ] Find the system range, apply soft limits
- [ ] Get control
	- [ ] Determine the system Gravity feed-forward value
	- [ ] Create a PID controller
	- [ ] Tune the PID to an acceptable level for control
- [ ] Create a default command that holds the system at the current height
- [ ] Create a setHeight function that takes a height, and returns a command that runs indefinitely to the target height
- [ ] Create a Trigger that indicates if the system is within a suitable tolerance of the commanded height.
- [ ] Bind several target positions to a controller
- [ ] Create a small auto sequence that moves to multiple positions in sequence.