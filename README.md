# Control theory - Introduction

A controller is a system that manages the behavior of other devices (or systems) using control loops. A control can be binary (on/off, in a thermostate for example), or have more complex values (adding gas to a car).

The theory behind continuosly operating dynamic systems is called control theory, and has as an objective to develp a control model for controlling dynamical systems. In general, we want the control to be stable, optimal and robust.

A feedback control loop happens when the control output depends on some error from some measurement of the dynamical system we are trying to control. For example, if we have an inverted pendulum, we want to mantain the pendulum in an upright position by moving the vehicle the pendulum is on. The error can be the angle difference between the upright and the current position of the pendulum. The higher the error, the more control signal.


## PID Feedback control

PID feedback control is a control loop using different types of errors to scale the control signal. PID stands for proportional-integral-derivative, and basically adds a control proportional to three errors:

- Proportional error: difference between an expected value and current value.
- Integral error: sum of all previous proportional errors.
- Derivative error: change of error with time.

For the current project, I simplified the values of the integral (since we have discrete increments of time) and differential error. For the integral error, I am also ignoring the integral windup given the limited amount of time the project is running; there are different solutions (only integrate in a certain window of time, reset the value and disable the integral function, ...). For the differential error the simplification is assuming that the derivative is the difference between current and past error. 

As a note, the derivative term provides damping of the control signal (to prevent unstability from the proportional term) and the integral term eliminates offset created by the proportional term.

In the current project, the controller for the steering function was implemented as a PID controller, while the throttle controller was implemented as a PD controller, since the cummulative error can affect negatively the speed of the car (unless we bound the time or value of the integral error).

A few considerations were made:

- Steering value is bounded to [-1, 1].
- Since I am not resetting the integral error, if the track of the simulator is more prone to turn to one direction (left more often than right), the error will tend to be towards the right side (overshoots positively). Therefore, the correction when the car is on the left side is slower (integral term is still positive), until the value is corrected.
- I added a second controller to try to speed up the velocity. I did not use any other signal than the error with respect the center of the road (we could potentially consider speed and angle). The result is that the car reaches up to 70mph with the current configuration. 

## Hyperparameter estimation

This is the area where I see more improvement in the project. The parameters were chosen based on experimentation, given the nature of the project. For the steering PID controller, the parameters are [0.06, 0.002, 2.0] for proportional, integral and derivative error. Increasing the proportional parameter increases the oscillations (specially with high speed, when becomes unstable!). Increasing the integrative parameter would make the changes of directions more difficult (since the integral error might not be bounded if we overshoot or undershoot, specially with higher speed and turning most of the time in one direction.

For the throttle error, we can have a higher proportional error to increase speeding up when the error becomes low. and slowing down faster when the error increases (controlling the oscillation with the derivative term). This is the main reasoning for having a higher proportional parameter.

Now, if I were to implement a more complex decission, probably we can setup some adaptive control system where the speed is also taken into consideration (to ensure the throttle is not overtaking when the error is higher and we reduce the velocity (to avoid overshooting on tight corners).  If we want to reach a good compromise with some other optimization method (twiddle or SGD) I would probably start by optimizing for minimizing steering errors (p, i and d) with constant velocity. Then try to optimize throttle with the constant parameters previously computed, to finalize with optimizing both at the same time using the starting values previously computed.

## Compiling and executing the project

- Clone the repo and cd to it on a Terminal.
- Create the build directory: `mkdir build`.
- `cd build`
- `cmake ..`
- `make`: This will create the executable `pid`, that contains a particle filter implementation implementation.

`pid` communicates with a Udacity simulator that reads the input data and produces estimations for the object. See [Udacity's seed project](https://github.com/udacity/CarND-Kidnapped-Vehicle-Project) for more information on how to install the simulator.

Open the simulator and run `./project-filter` to enable the connection. Select Project 4 from the simulator.

