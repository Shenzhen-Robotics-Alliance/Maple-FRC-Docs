# PID in Robotics

Created by: 亦然 刘
Created time: June 5, 2024 3:42 AM
Tags: Basic

## What is PID?

> PID stands for Proportional-Integral-Derivative, is a type of control system that can maintain a mechanism at the desired position or level.
> 

Watch ▶️[this video](https://www.bilibili.com/video/BV1et4y1i7Gm) to know more.

## Where is it used?

A mechanism needs to have two features to use a PID controller. 

1. an output (pneumatics, motors, etc.)
2. an input (encoder, sensors, etc.)

As long as the system have these two above, you can use a PID-like controller to move it to a set point.

## Using EasyPIDController.java

EasyPIDController.java is a utility that helps you write your first PID Controller with very easy configurations.

Let’s learn to use it by going over a basic example: controlling the rotation of a robot.

The first step is to read the rotation, which we have just learned from the previous course.

under frc.robot.Utils.Tests, create a new class named `ChassisHeadingPIDControllerTest` , which implements  `SimpleRobotTest` 

```java
package frc.robot.Utils.Tests;

public class ChassisHeadingPIDControllerTest implements SimpleRobotTest {
	@Override
	public void testStart() {
	
	}

	@Override
  public void testPeriodic() {

  }
}
```

Now let’s declare a chassis module and a gyro

```java
	private final ChassisModule chassis;
	private final SimpleGyro gyro;
	public ChassisHeadingPIDControllerTest(ChassisModule chassis, SimpleGyro gyro) {
		this.chassis = chassis;
		this.gyro = gyro;
	}
	
	@Override
	public void testStart() {
		chassis.init();
		chassis.reset();
	  chassis.enable();
	  gyro.reset();
	}
```

Next, create an EasyPIDController instance with its profile, it takes the following params:

- `*maximumPower` the maximum amount of output power allowed.*
- `*errorStartDecelerate` The amount of error at which the mechanism start decelerates.  When the mechanism is moving to the desired position, it will first move at full power and start to slow down when the error goes under this value.*
- `*minimumPower` the minimum amount of power to make the mechanism move*
- `*errorTolerance` the amount of error to ignore, if the error is under this tolerance, the mechanism will ignore it and stay still*
- `*errorAsMechanismInPlace` the amount of error that the mechanism accepts and thinks that the mechanism is in place when calling the method `controller.*isMechanismInPosition()`

```java
	private final EasyPIDController controller = new EasyPIDController(
		new EasyPIDController.EasyPIDProfile(
			0.7, // limit the chassis motor power to 70%
		  Math.toRadians(60), // the chassis starts slowing down when the error is smaller than 90 degrees
		  0.05, // to move the chassis, it takes at least 5% the motor power
		  Math.toRadians(2), // if the error is within 2 degrees, we ignore it
		  Math.toRadians(5), // if the error is wihtin 5 degrees, we think the chassis is in position
		  0.3, // the chassis needs 0.3 seconds to slow down
		  true // when the robot turns 360 degrees, it goes back to 0 degrees.  we call this "in cycle"
		), 0 // the chassis starts at zero degrees
	);
```

Now let’s test our controller

```java
	/* use gamepad at port 1 to control */
	private final XboxController xboxController = new XboxController(1);
	
	@Override
	public void testPeriodic() {
		/* we will try make the chassis stay at its starting rotation */
		controller.setDesiredPosition(0);
		/* if button A is pressed, send the controller's output to chassis as rotating power */
		if (xboxController.getAButton())
			chassis.setTurn(
				/* feed the gyro's data to the PID controller */
				controller.getMotorPower(gyro.getYawVelocity(), gyro.getYaw())
			, null);
		/* otherwise, we use the controller's righter stick to control the rotation manually */
		else
			chassis.setTurn(-xboxController.getRightX(), null);
		chassis.periodic();
	}
```