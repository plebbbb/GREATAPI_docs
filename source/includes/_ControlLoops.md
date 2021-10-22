# Control Loops

GREATAPI provides PID control loops to assist you in developing PID control systems for your robot. 

## Proportional, Integral, and Derivative controllers

The controllers GREATAPI offers are separate P, I, and D controllers throught the <code>controlelement</code> class. 
Each of the three controllers have their own child class of controlelement. 

Class Name |
---------- |
Proportional |
Integral |
Derivative |

<code>controlelement</code> child classes provide one function to the user: 

Function | Parameters | Purpose |
-------- | ---------- | ------- |
compute(double target, double current) | Target value and current value | Returns the corresponding component of the control loop |

<aside class = "warning">
Note that <code>controlelement</code> itself does not compute any outputs. Please contruct one of the available child classes instead when programming your robot.  
</aside>

### Proportional controller

## Unified Control Loop

> Prototype

```
//Modular control loop, computes values for a set of control elements
struct control_loop{
    std::vector<controlelement*> elementset;
    double maxcap;
    double mincap;
    //if you dont like caps just set them really high, like +-INT_MAX or something
    control_loop(std::vector<controlelement*> val, std::pair<double,double> caps):elementset(val){
        maxcap = std::get<0>(caps);
        mincap = std::get<1>(caps);
    }
    double update(double target, double current){
        double returnval = 0;
        //no enhanced for to minimize risk of sketchy copying issues
        for (int i = 0; i < elementset.size(); i++){
            returnval += elementset[i]->compute(target,current);
        }
        return (returnval <= maxcap) ? ((returnval >= mincap) ? returnval : mincap) : maxcap;
    }
};
```

> Example

```
//create the new controlelements
greatapi::controlelement *PPos = new greatapi::Proportional(20.0, std::pair(__DBL_MAX__, __DBL_MIN__));
greatapi::controlelement *IPos = new greatapi::Integral(0, std::pair(__DBL_MAX__, __DBL_MIN__));
greatapi::controlelement *DPos = new greatapi::Derivative(0, std::pair(__DBL_MAX__, __DBL_MIN__));

//add the controlements to a new std::vector
std::vector<greatapi::controlelement *> PIDPosElements;
PIDPosElements.push_back(PPos);
PIDPosElements.push_back(IPos);
PIDPosElements.push_back(DPos);

//Create the control_loop unified control loop
greatapi::control_loop PIDPos(PIDPosElements, std::pair(INT32_MAX, INT32_MIN));
```


The Unified Control Loop computes the final outputs for a PID controller with any amount of P, I, and D <code>controllerelements</code> by summing their outputs together.

