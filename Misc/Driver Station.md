---
tags:
  - stub
aliases:
  - Controllers
---

The DriverStation software handles communication between your laptop and the FRC robot. 

![[driverstation.jpg]]

Official documentation is here:
https://docs.wpilib.org/en/stable/docs/software/driverstation/index.html
#### Driver Interface
The first tab is the Driver interface. This handles 
- Enabling the robot: When enabled, motors can actually spin, and the robot can be used
- Disabling the robot: This halts all output, and stops some running code processes.
- Mode selection (Teleop, Autonomous, Test, and Practice)
- Displays basic diagnostics (battery states, status, and communication status)
#### Diagnostic tab 
This provides some network and version info. Sometimes helpful for troubleshooting. It is also where we can restart the RIO or robot code if we need to.

![[driverstation-diagnostic-tab.png]]

#### Configuration Tab
This tab allows us to set our team number and dashboard. We don't use it very often.

![[driverstation-configuration-tab.png]]

>[!WARNING] Team Number
>If the team number is not set correctly, you will not be able to run the robot. If you are not able to enable the robot and have looked at everything else, check this
#### USB Tab
This tab lists connected controllers and joysticks, and allows you to rearrange them.

Connected joysticks are given a "Port" from 0 to 5; The connected controller needs to match the Port expected by the code, and this is done here. 

This will also allow you to select devices, and see the 
#### :luc_zap: Tab
This tab gives some readings on the robot's hardware, but we don't actually use this tab for debugging.

![[driverstation-lightning-tab.png]]

#### Log Pane
The right side of the DriverStation always shows printed messages, warnings, and errors. These messages can also be seen in the RioLog inside VSCode.

Clicking  the ⚙️ Icon above the text display can let you filter this display; Unfortunately several libraries and built-in functions are still very spammy, making this small interface mostly useless for debugging.

With a bit of hunting, you can access the Log Viewer; This allows you to read logs saved from prior matches or practice sessions. These can be very useful for some diagnostic faults, especially during matches where replicating or identifying conditions in other ways can be difficult. 
https://docs.wpilib.org/en/stable/docs/software/driverstation/driver-station-log-viewer.html
## Controllers + Inputs

If you click the USB Tab, you can interact and view configured controllers. 

![[driverstation-usb-tab.png]]

When working with a new controller, it's helpful to verify that it's detected in the right port, and that the joystick axes and buttons are numbered the way you expect.

The button and joystick numbering differ between similar looking controllers, so it's typically best to use `CommandXboxController` , which offer a way to access buttons by name; This makes them more robust and easier to identify without counting on this interface. This lets you access axes as `stick.getLeftX()`, `stick.getRightY()`, `stick.getLeftTrigger()`, `stick.a()`, etc.

When you do use numbers, just double check on the interface before deployment. 
<br>

> [!TIP] Team Controller Conventions
> Because we're often sharing the same code and laptops across many teammates and projects, one thing we *really* want to avoid is having controllers not work or worse, start launching unexpected actions. 
> Due to this, we have a convention:
> - Driver Xbox/Playstation controller on Port 0 
> - Operator/secondary driver controller on Port 1
> - If you need to add a third, separate controller for special cases, use 2-5. This is rare.
>   
> This ensures everyone's DriverStation and joystick ports are the same.

<br>

> [!WARNING] X-input and D-Input modes
> Many controllers have two different modes, indicated with a slider on the back. 
> One is X-input, following the Xbox Controller numbering order. 
> The other is D-input, following the HID Joystick standards.
> 
> These modes are often similar, but not guaranteed to be identical. The subtle  differences in button order can be very confusing. A common difference is the Left/Right Triggers changing from Analog (showing up as a Joystick Axis) to Digital (show up as a button).
> 
> Make sure you select the appropriate mode (usually X-input mode) early, make a comment, and double check if something seems off.

