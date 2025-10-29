---
tags:
  - stub
---

## Success Criteria
- [ ] Run a motor using
- [ ] Plot the motor's position over time
- [ ] Configure a motor's ID and basic settings
- [ ] View an absolute encoder's output



## Introduction to the Hardware Client
The Rev hardware client allows us to configure and update rev hardware such as motor controllers.
Official Documentation here:
https://docs.revrobotics.com/rev-hardware-client/gs/navigation
![[rev-hardware-client.png]]


### Configuring a Spark
After selecting a device from the connected hardware, you can access the configuration settings. We mostly use this to set the CAN id, as we configure the rest in code.
https://docs.revrobotics.com/rev-hardware-client/ion/spark-max
![[rev-hardware-config.png]]


## Identifying a motor ID
- Plug in your computer into any available Spark controller. The one you're interested is easier.
- If you don't see anything useful, click "Scan For Devices" down in the bottom left
- You should see one (or more!) devices in the left side.
- Select the device that looks like your motor
- Click the "Blink" button, and look at your physical hardware. The light should be flickering
- If not, repeat this process until you find it.
- You can see the Device ID in the device settings, and put that in your code. 



## Rev Reference Snippets


> [!BUG] Config import warning!
> Watch your imports when setting up Spark Configs. If you're not mindful, VSCode will pull in 
> `import com.revrobotics.servohub.ServoHub.ResetMode;`
> instead of the desired 
> `import com.revrobotics.spark.SparkBase.ResetMode;`
> This shows up as a error on your line for 
>  `motor.config(....);`




###### Configuration Parameters
```java
motor.configure(
	config, 
	//This determines if we reset all existing parameters. 
	//Generally, we always want this on initialization, and then 
	//Manually set the values we expect to see for the robot.
	//The typical case for kNoResetSafeParameters is when we
	//change configuration after bootup. Changing Coast/Brake mode
	//is a common example.
	ResetMode.kResetSafeParameters, 
	
	//This option determines whether this config's settings
	//are saved on the Spark when it reboots.
	//In most cases, this doesn't matter, since the robot code
	//and Spark usually boot at the same time. However, in competition,
	//there's cases where a spark might lose power and reboot by itself!
	//Because of this, you usually want to set it to kPersistParameters,
	//but there's certain cases where kNoPersistParameters is preferable,
	//typically when it relates to increasing output power after validating
	//the system state.
	PersistMode.kPersistParameters
);
```



