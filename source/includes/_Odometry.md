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

<aside class="aside">
<code>TWheel_odom_rotation</code> uses distance measurements from <code>TWheel</code> objects. These are described in the <code>TWheel</code> section of the site. Wheel diameters are defined during the construction of <TWheel> objects, not in <code>TWheel_odom_rotation</code>
</aside>
  
<aside class = "warning">
<code>TWheel_odom_rotation</code> must use two parallel wheel encoders.
</aside>

<code>odom_rotation</code> provides two functions for the end user:

Function | Purpose | Output |
-------- | ------- | ------ |
get_heading() | Returns absolute heading of robot from starting position | <code>SRAD</code> angle object
applyOffset(SRAD real) | Calibrates odometry output to match that of the real angle | nothing 

It is recommended to 


### IMU_odom_rotation

<code>IMU_odom_rotation</code> is a basic wrapper for the vex IMU. 

The only capability it provides is to adjust for 

> Prototype
```cpp
  IMU_odom_rotation(int port, double driftcompensationfac)
```

> Example
```cpp
  greatapi::IMU_odom_rotation example(15, 1.01); //IMU on port 15, 101% drift compensation factor
```
