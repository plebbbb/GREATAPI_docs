# Odometry

GREATAPI provides multiple feature levels of odometry for different robot configurations.

## Rotational Odometry

The lowest feature level GREATAPI provides is rotation tracking via the <code>odom_rotation</code> class.

Various tracking approaches are supported by child classes of <code>odom_rotation</code>

Class Name | Utlized Sensors | Required Measurements |
---------- | --------------- | --------------------- |
IMU_odom_rotation | 1x V5 Inertial Sensor | Inertial drift compensation multiplier
DoubleIMU_odom_rotation | 2x V5 Inertial Sensor | none
TWheel_odom_rotation | 2x Wheel encoders(any type) | Distance between wheel encoders, wheel diameters
  
<aside class = "warning">
<code>TWheel_odom_rotation</code> has unique and specific restrictions. Please go to its section for more details.
</aside>

<code>odom_rotation</code> child classes provide two functions for the end user:

Function | Purpose | Output |
-------- | ------- | ------ |
get_heading() | Returns absolute heading of robot from starting position | <code>SRAD</code> angle object
applyOffset(SRAD real) | Calibrates odometry output to match that of the real angle | nothing 

<aside class = "warning"> Note that <code>odom_rotation</code> itself does not provide any angle calculation capabilities. Please construct one of the availiable child class options instead when programming your robot.
</aside>

### IMU_odom_rotation

> Prototype
  
```cpp
  IMU_odom_rotation(int port, double driftcompensationfac)
```

> Example
  
```cpp
  greatapi::odometry::IMU_odom_rotation example(15, 1.01); //IMU on port 15, 101% drift compensation factor
  while(true){
    greatapi::SRAD currentangle = example.get_heading(); //get current heading
    cout << currentangle; //print angle on terminal
    pros::delay(10); //10ms delay
  }
```

> The v5 IMU has an update period of 10ms. Checking more frequently than that will not yield different results.
  
<code>IMU_odom_rotation</code> is a basic wrapper for the vex IMU. 

It provides the capability to counteract integration drift via an automatic scaling the calculated angle by a specific amount. 

### Integration drift

Integration drift is a phenomenon commonly found in inertial sensors. Inertial sensors measure the acceleration, and angular velocity of the robot at any point in time. To convert these values into more useful measurements like speed, heading, or position, we must use a technique known as integration. While we don't know where the robot actually is with these sensor values, we can assume that it starts at some fixed state(like not moving). From there, we can convert our accelerations and angular velocities into velocity, position, and heading changes. These can be added to the previous calculated position to get the current position.

The problem with this approach is obvious - Inaccuracies cannot be removed, and can add up until the sensor is inaccurate.

<alert class = 'notice'> 
Incidentally, this is why most vex libraries don't provide the feature to calculate position using the IMU. To get position data, one needs to compound in two layers(acceleration -> velocity -> position), meaning that any inaccuracy in velocity will indefinitely increase inaccuracy in position. Heading data only needs one compounding step(angular velocity -> heading), so this infinite compounding situation doesn't occur.
</alert>

A quick and quite dirty way of compensating for this is to just scale the total degrees traveled by a hard set multiplier. One can experimentally determine a multiplier just by spinning the robot a known amount of degrees, and then dividing the known value by the sensor reported angle. Spinning over 10 rotations is suggested to minimize inaccuracy in determining the real traveled angle of the robot.

The multiplier is to be put in as the <code>driftcompensationfac</code> parameter of the constructor.


### DoubleIMU_odom_rotation

Supposedly, there is a method of canceling out inertial drift with the same model of sensor by flipping one over and averaging out their outputs. Somehow, the drift direction of each sensor would be opposite to each other, which leads to a rough canceling out of them due to the same sensor hardware being used. There are specifics to the implementation that I am not clear on, but it is a possiblity to be added in future versions.


### TWheel_odom_rotation

> Prototype
  
```cpp
  TWheel_odom_rotation(TWheel* L, TWheel* R, double dist_btwn)
```

> Example
  
```cpp
  //TWheel is an abstract class. You should be using the constructor of specific TWheels
  greatapi::TWheel* leftwheel = new greatapi::TWheel_Motor(5, pros::E_MOTOR_GEARSET_18,true, 2.75); //V5 motor, 200RPM, reversed, 2.75in wheel

  //For information on TWheel, please see the TWheel section of the site.
  greatapi::TWheel* rightwheel = new greatapi::TWheel_RotationSensor(4, false, 4); //V5 rotation sensor, not reversed, 4in wheel
  //note that we are making TWheel* (TWheel pointers), not TWheels.
 
  greatapi::odometry::TWheel_odom_rotation example = *new greatapi::odometry::TWheel_odom_rotation(leftwheel,rightwheel,15) //15 inches between

  while(true){
    greatapi::SRAD currentangle = example.get_heading(); //get current heading
    cout << currentangle; //print angle on terminal
    pros::delay(10); //10ms delay
  }
```
  
<code>TWheel_odom_rotation</code> provides rotational odometry through usage of two parallel wheels mounted onto some type of encoder.

<aside class = 'warning'>
TWheel objects must be constructed with the new keyword in order to ensure that they exist in memory as the class doesnt store them. The constructor takes TWheels in pointer form, meaning that you need to make TWheel* pointers instead of TWheels. The positive direction of travel must be forwards in your TWheel constructors.
</aside>

By comparing the traveled distances between two parallel wheels, it is possible to determine the relative distance from their starting orientations. We can logically explain this by noting that two parallel wheels will always traverse the same distance in translations. Therefore, any difference in their travel must be from rotations. 

The mathematical expression to determine this distance is the following:

**Angle from start(in radians) = (distance of right wheel - distance of left wheel) / (distance between wheels)** 

This yields positive angles for CCW rotations, and negative angles for CW rotations.

<aside class = 'notice'>
The wheels can be mounted anywhere on the robot, even asymetrically, as long as they are parallel. The <code>dist_btwn</code> parameter needs the distance between the wheels only in the axis perpendicular to the travel direction of the wheel.
</aside>

## Positional Odometry

GREATAPI also provides the capability to determine the absolute position of a robot in all relevant axises for a vex game(X, Y, and heading)


> Prototype
  
```cpp
  greatapi::odometry::odometry(TWheel* X, distance X_to_ctr, TWheel* Y, distance Y_to_ctr, odom_rotation* rotation)
```

> Example
  
```cpp
	//TWheel is an abstract class. You should be using the constructor of specific TWheels
	greatapi::TWheel* leftwheel = new greatapi::TWheel_Motor(5, pros::E_MOTOR_GEARSET_18,true, 2.75); //V5 motor, 200RPM, reversed, 2.75in wheel

	//For information on TWheel, please see the TWheel section of the site.
	greatapi::TWheel* rightwheel = new greatapi::TWheel_RotationSensor(4, false, 4); //V5 rotation sensor, not reversed, 4in wheel
	//note that we are making TWheel* (TWheel pointers), not TWheels.

	greatapi::TWheel* rearwheel = new greatapi::TWheel_ADIEncoder('A','B',false,2.75); //V4 rotation encoder, not reversed, 2.75in wheel

	greatapi::odometry::TWheel_odom_rotation example = *new greatapi::odometry::TWheel_odom_rotation(leftwheel,rightwheel,15); //15 inches between the wheels

	greatapi::odometry::odometry test(leftwheel, greatapi::inches(7.5), rearwheel, greatapi::inches(8), &example); 
  //left wheel is 7.5in from center of rotation, rear wheel is 8 inches away.

  greatapi::position location(greatapi::coord(0,0),SRAD(0)); //the position object storing the location of our robot
	
  //this loop is probably the update pattern you would want to use for your projects.
  while(true){ 
		location = test.calculateposition(location);
    cout << location.x << " " << location.y << " " << location.angle; //print X, Y and angle after each test
    pros::delay(10); //10ms wait before updating position.
	}
```
  

### Configuration
Positional odometry requires a set of two tracking wheels, positioned 90 degrees away from each other, as well as any valid <code>odom_rotation</code> object. In addition, the distance of each tracking wheel from the center of rotation of the robot is required.

<aside class = "notice">
The same wheel encoder can be used for both <code>odom_rotation</code> and <code>odometry</code>. A pure wheel encoder setup only needs 3 wheel encoders.
</aside>

#### TBD: ADD IMAGE HERE TO EXPLAIN CONFIG

<code>odom_rotation</code> child classes provide a single function for the end user:

Function | Purpose | Output |
-------- | ------- | ------ |
calculateposition(position initial) | Returns absolute position and heading of robot | <code>position</code> object

#### TBD: EXPLAIN HOW IT WORKS
