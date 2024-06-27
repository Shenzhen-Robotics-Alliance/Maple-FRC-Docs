# The Maple🍁 Style of Coding

Created by: 亦然 刘
Created time: June 5, 2024 3:29 AM
Tags: Basic

## TO OHTER TEAMS

- This document is only about the style that we (Iron Maple) write our code.  It does not discuss which way of coding is better, nor does it suggest that anyone should switch to our way of coding.  Every programming styles have their own advantages, it is recommended for everyone to develop their own styles of programming.
- The principal is to achieve advanced function while making code as short, simple and easy-to-understand as possible.
- To Achieve this, we did not use [“Command-Based Programming”](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html) that WPILIB suggests.  We discarded the concept of “command”.  Instead, we use “state-machines” style.  This means that for those already familiar with FRC Programming, it might take some time to understand our coding logic.
- Also, we use custom libraries that covers linear algebra, closed-loop controls, swerve kinematics, robot configurations and physics simulation.  These libraries are

## FOR THOSE IN OUR TEAM

- This document is about the style of our code.  Every member is required to learn and follow this style, it is the key to maintaining an elegant and comfortable development process.
- Code under Utils are custom libraries written by your seniors.  It provides helper functions and makes your lives easier.  If you don’t want to spend hours dealing with crazy math formulas, LEARN ABOUT THESE LIBRARIES.

## State-Machine Style

Command-Based Style

![屏幕截图 2024-06-13 084344.png](The%20Maple%F0%9F%8D%81%20Style%20of%20Coding%20639c7b8d9f0748c1a191f75358152093/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-06-13_084344.png)

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

State-Machine Style

![屏幕截图 2024-06-13 084201.png](The%20Maple%F0%9F%8D%81%20Style%20of%20Coding%20639c7b8d9f0748c1a191f75358152093/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-06-13_084201.png)

```java
// ArmModule.java
public class ArmModule extends RobotModuleBase {
 ...
 @Override
 protected void periodic(double dt) {
	 switch (currentStatus) {
		 case MOVING_TO_LOW_POS -> ...;
		 case MOVING_TO_HIGH_POS -> ...;
		 case HOLDING_AT_LOW_POS -> ...;
		 case HOLDING_AT_HIGH_POS -> ...;
	 }
	}
	
	public void raiseArm(RobotModuleOperatorMarker operator) {
		if (!super.isOwner(operator)) return;
		this.currentStatus = MOVING_TO_HIGH_POS;
	}
	public void lowerArm(RobotModuleOperatorMarker operator) {
		if (!super.isOwner(operator)) return;
		this.currentStatus = MOVING_TO_LOW_POS;
	}
}
// ShootingService.java
public class ShootingService extends RobotServiceBase {
	protected void periodic(double dt) {
		if (pilotController.keyOnPressed(robotConfig.getConfig("controls/raiseArmButtonID")) 
			arm.raiseArm(this);
		else if (pilotController.keyOnPressed(robotConfig.getConfig("controls/lowerArmButtonID")) 
			arm.lowerArm(this);
	}
}
```

## Program Structure

- **Drivers** Code that communicates with motors, encoders, gyros and so on. We stratified the functions of a type of hardware into [interfaces](https://www.w3schools.com/java/java_interface.asp) and created different implements for different [COTS](https://docs.wpilib.org/en/stable/docs/software/frc-glossary.html#term-COTS).  For example, `TalonFXMotor` implements both the `Motor` and `Encoder` interface, while `CanCoder` only implements the `Encoder` interface.  This way we can switch hardware without modifying the code.
- **Modules** Code that controls a specific module on the robot, like `ShooterModule`, `ArmModule`, `ChassisModule` and so on.  All the classes here inherits `RobotModuleBase`  Notice that this part of code only includes the controlling algorithms and kinematics.  For example, in `ArmModule` , we only work on “how to move the arm up”.  We do not decide “when to move the arm up” and leave the two APIs `raiseArm(RobotModuleOperatorMarker operator)` and `lowerArm(RobotModuleOperatorMarker operator)` for **Services**.
- **Services** Code that controls the operation logic of a function on the drivers’ gamepad.  All the classes here inherits `RobotServiceBase`, which inherits `RobotModuleOperatorMarker` .  This layer of code is the link between the pilot and the **Modules**, it reads input from the drivers’ gamepad and calls to the APIs in the Modules by: `raiseArm(this)` .

## Never-Nester

Let’s take a look at too pieces of codes:

```java
private void readWheelConfigurationXMLFile(String filePath) {
  try {
    // Read each wheel calibration
    String[] wheels = {"frontLeft", "frontRight", "backLeft", "backRight"};
    for (String wheel : wheels) {
	    NodeList nodeList = rootElement.getElementsByTagName(wheel);
	    if (nodeList.getLength() > 0) {
		    Node wheelNode = nodeList.item(0);
		    if (wheelNode.getNodeType() == Node.ELEMENT_NODE) {
			    Element wheelElement = (Element) wheelNode;
				  Map<String, String> configMap = new HashMap<>();
			    NodeList configNodes = wheelElement.getChildNodes();
			    for (int i = 0; i < configNodes.getLength(); i++) {
				    Node configNode = configNodes.item(i);
				    if (configNode.getNodeType() == Node.ELEMENT_NODE) {
					    Element configElement = (Element) configNode;
						  configMap.put(
							  configElement.getTagName(), 
							  configElement.getTextContent()
							);
						}
					}
					calibrationData.put(wheel, configMap);
				}
			}
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

It looks nasty, how do we fix it?

> **Trick2**: extract long for loop body to a separate method
> 

```java
private void readWheelsConfigurationFromXMLFile(String filePath) {
  try {
    // Read each wheel calibration
    String[] wheels = {"frontLeft", "frontRight", "backLeft", "backRight"};
    for (String wheel : wheels) 
	    readWheelConfig(wheel)
	} catch (Exception e) {
		e.printStackTrace();
	}
}

private void readSingleWheelConfigs(String wheelName) {
	NodeList nodeList = rootElement.getElementsByTagName(wheel);
	if (nodeList.getLength() > 0) {
		Node wheelNode = nodeList.item(0);
		if (wheelNode.getNodeType() == Node.ELEMENT_NODE) {
			Element wheelElement = (Element) wheelNode;
			Map<String, String> configMap = new HashMap<>();
			NodeList configNodes = wheelElement.getChildNodes();
			for (int i = 0; i < configNodes.getLength(); i++) {
				Node configNode = configNodes.item(i);
				if (configNode.getNodeType() == Node.ELEMENT_NODE) {
					Element configElement = (Element) configNode;
					configMap.put(configElement.getTagName(), configElement.getTextContent());
				}
			}
			calibrationData.put(wheel, configMap);
		}
	}
}
```

> **Trick 2**: process special situations first
> 

```java
/** 
* reads the configuration of a single wheel
* @param wheelName the name of the wheel
*/
private void readSingleWheelConfigs(String wheelName) {
	NodeList nodeList = rootElement.getElementsByTagName(wheel);
	if (nodeList.isEmpty())
		return; // skit the rest of the code
	final Node wheelNode = nodeList.item(0);
	if (wheelNode.getNodeType() != Node.ELEMENT_NODE)
		return; // skip the rest of the code
	
	final Element wheelElement = (Element) wheelNode;
	Map<String, String> configMap = new HashMap<>();
	NodeList configNodes = wheelElement.getChildNodes();
	for (int i = 0; i < configNodes.getLength(); i++) {
		Node configNode = configNodes.item(i);
		if (configNode.getNodeType() == Node.ELEMENT_NODE) {
			Element configElement = (Element) configNode;
			configMap.put(configElement.getTagName(), configElement.getTextContent());
		}
	}
	calibrationData.put(wheel, configMap);
}
```

(Repeat Trick 1 and 2 again)

```java
/** 
* reads the configuration of a single wheel
* @param wheelName the name of the wheel
*/
private void readSingleWheelConfigs(String wheelName) {
	...
	final Element wheelElement = (Element) wheelNode;
	Map<String, String> configMap = new HashMap<>();
	NodeList configNodes = wheelElement.getChildNodes();
	for (Node configNode:configNodes) 
		addConfigToMap(configNode, configMap);
	calibrationData.put(wheel, configMap);
}

private void addConfigToMap(Node configNode, Map<String, String> configMap) {
	if (configNode.getNodeType() != Node.ELEMENT_NODE) return; // skip the rest
	final Element configElement = (Element) configNode;
	configMap.put(configElement.getTagName(), configElement.getTextContent());
}
```

Let’s compare the two code

<aside>
😩 Before

</aside>

```java
private void readWheelConfigurationXMLFile(String filePath) {
  try {
    // Read each wheel calibration
    String[] wheels = {"frontLeft", "frontRight", "backLeft", "backRight"};
    for (String wheel : wheels) {
	    NodeList nodeList = rootElement.getElementsByTagName(wheel);
	    if (nodeList.getLength() > 0) {
		    Node wheelNode = nodeList.item(0);
		    if (wheelNode.getNodeType() == Node.ELEMENT_NODE) {
			    Element wheelElement = (Element) wheelNode;
				  Map<String, String> configMap = new HashMap<>();
			    NodeList configNodes = wheelElement.getChildNodes();
			    for (int i = 0; i < configNodes.getLength(); i++) {
				    Node configNode = configNodes.item(i);
				    if (configNode.getNodeType() == Node.ELEMENT_NODE) {
					    Element configElement = (Element) configNode;
						  configMap.put(
							  configElement.getTagName(), 
							  configElement.getTextContent()
							);
						}
					}
					calibrationData.put(wheel, configMap);
				}
			}
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

<aside>
😀 After

</aside>

```java
private void readWheelsConfigurationFromXMLFile(String filePath) {
  try {
    // Read each wheel calibration
    String[] wheels = {"frontLeft", "frontRight", "backLeft", "backRight"};
    for (String wheel : wheels) 
	    readWheelConfig(wheel)
	} catch (Exception e) {
		e.printStackTrace();
	}
}

private void readSingleWheelConfigs(String wheelName) {
	NodeList nodeList = rootElement.getElementsByTagName(wheel);
	if (nodeList.isEmpty()) return;
	
	final Node wheelNode = nodeList.item(0);
	if (wheelNode.getNodeType() != Node.ELEMENT_NODE) return;
	
	final Element wheelElement = (Element) wheelNode;
	Map<String, String> configMap = new HashMap<>();
	NodeList configNodes = wheelElement.getChildNodes();
	for (Node configNode:configNodes) 
		addConfigToMap(configNode, configMap);
	
	calibrationData.put(wheel, configMap);
}

private void addConfigToMap(Node configNode, Map<String, String> configMap) {
	if (configNode.getNodeType() != Node.ELEMENT_NODE) return;
	final Element configElement = (Element) configNode;
	configMap.put(configElement.getTagName(), configElement.getTextContent());
}
```