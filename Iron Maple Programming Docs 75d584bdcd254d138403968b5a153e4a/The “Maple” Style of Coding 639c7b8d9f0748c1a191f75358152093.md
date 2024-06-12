# The “Maple” Style of Coding

Created by: 亦然 刘
Created time: June 5, 2024 3:29 AM
Tags: Basic

## TO OHTER TEAMS

- This document is only about the style that we (Iron Maple) write our code.  This document does not discuss which way of coding is better, nor does it suggest that anyone should switch to our way of coding.  Every programming styles have their own advantages, it is recommended for teams to develop their own styles of programming.
- The principal is to achieve advanced function while making code as short, simple and easy-to-understand as possible.
- To Achieve this, we did not use the [“Command-Based Programming”](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html) that WPILIB suggests.  We discarded the concept of “command”.  Instead, we put all the code into run-loops.  Every subsystem of our robot updates 101 iterations every second, and do what they are suppose to.   This means that for those already familiar with FRC Programming, it might take some time to understand our code.
- Also, we use custom libraries that covers linear algebra, closed-loop controls, swerve kinematics, robot configurations and physics simulation.  These libraries are

## FOR OUR MEMBERS

- This document is about the style of our code.  Every member is required to learn and practice this style, this is the key to maintaining a smooth and comfortable developing process.
- Code under Utils are custom libraries written by your seniors.  It provides convenient functions and makes your lives easier.  So, if you don’t want to spend hours dealing with crazy math formulas, know what these libraries do.

## Run-Loop-Based Programming

Command-Based Programming

![Untitled](The%20%E2%80%9CMaple%E2%80%9D%20Style%20of%20Coding%20639c7b8d9f0748c1a191f75358152093/Untitled.png)

```java
// Arm.java
public class Arm extends PIDSubsystem {
	public Command raiseArm() {
		return Command.sequence(
			...
		);
	}
	public Command lowerArm() {
		return Command.sequence(
			...
		);
	}
	
	@Override
  public void useOutput(double output, double setpoint) {
    ...
  }

  @Override
  public double getMeasurement() {
    ...
  }

  public boolean atSetpoint() {
    ...
  }
}

// RobotContainer.java
button.onTrue(Arm.raiseArm().endThen(Shooter.startShooter());
```

Run-Loop-Based Programming

```java
// ArmModule.java
public class Arm extends RobotModuleBase {
 ...
 @Override
 protected void periodic(double dt) {
	 switch (currentStatus) {
		 case MOVING_TO_LOW_POS -> ...;
		 case MOVING_TO_HIGH_POS -> ...;
		 case HOLDING_AT_LOW_POS -> ...;
		 case HOLDING_AT_HIGH_POS -> ...;
	 }
	 motor.setOutput(...);
	}
	public void raiseArm() {
		this.currentStatus = MOVING_TO_HIGH_POS;
	}
	
	public void lowerArm() {
		this.currentStatus = MOVING_TO_HIGH_POS;
	}
}
// ShootingSwervice.java
public class ShootingService extends RobotServiceBase {
	protected void periodic(double dt) {
		if (pilotController.keyOnPressed(robotConfig.getConfig("controls/raiseArmButtonID")) 
			arm.raiseArm();
		else if (pilotController.keyOnPressed(robotConfig.getConfig("controls/lowerArmButtonID")) 
			arm.lowerArm();
	}
}
```

## Program Structure

## Never-Nester