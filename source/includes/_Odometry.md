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

<aside class="notice">
<code>TWheel_odom_rotation</code> uses distance measurements from <code>TWheel</code> objects. These are described in the <code>TWheel</code> section of the site. Wheel diameters are defined during the construction of <TWheel> objects, not in <code>TWheel_odom_rotation</code>
</aside>
  
<aside class = "warning">
<code>TWheel_odom_rotation</code> must use two parallel wheel encoders.
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
    SRAD currentangle = example.get_heading(); //get current heading
    System.out.println(currentangle); //print angle on terminal
    pros::delay(10); //10ms delay
  }
```

> The v5 IMU has an update period of 10ms. Checking more frequently than that will not yield different results.
  
<code>IMU_odom_rotation</code> is a basic wrapper for the vex IMU. 

It provides the capability to counteract integration drift via scaling the calculated angle by a specific amount. 

### Integration drift
Integration drift is a phenomenon commonly found in inertial sensors. Inertial sensors measure the acceleration, and angular velocity of the robot at any point in time. To convert these values into more useful measurements like speed, heading, or position, we must use a technique known as integration. While we don't know where the robot actually is with these sensor values, we can assume that it starts at some fixed state(like not moving). From there, we can convert our accelerations and angular velocities into velocity, position, and heading changes. These can be added to the previous calculated position to get the current position.

There are two problems with this approach that form what we call integration drift.
  
* Calculation inaccuracies cannot be removed, and can(and will) add up until the sensor is very inaccurate
* The integration process of converting measurements upscales any sensor inaccuracies.
  * Incidentally, this is why most vex libraries don't provide the feature to calculate position using the IMU. To get position data, one needs to integrate twice(acceleration -> velocity -> position), greatly reducing accuracy. Heading data only needs one integration(angular velocity -> heading), which isn't as brutal.
