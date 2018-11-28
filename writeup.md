## Building a Controller

### Motor Commands

The implementation in the exercise notebook was used as a refererence. As it turns out the drone configuration was a little different in this project compared to the exercise.

The following two images show orientation and rotor enumeration:

(kindly provided by Mike Hahn in Slack https://udacity-flying-car.slack.com/archives/C919QLSGY/p1524949885000076)

![https://raw.githubusercontent.com/magnusja/FCND-Controls-CPP/master/drone2.png](https://raw.githubusercontent.com/magnusja/FCND-Controls-CPP/master/drone2.png)

![https://raw.githubusercontent.com/magnusja/FCND-Controls-CPP/master/drone1_rev2.jpg](https://raw.githubusercontent.com/magnusja/FCND-Controls-CPP/master/drone1_rev2.jpg)

This results in a different formular system to solve:

F_tot = F0 + F1 + F2 + F3
tau_x = (F0 - F1 + F2 - F3) * l                 // This is Roll
tau_y = (F0 + F1 - F2 - F3) * l                 // This is Pitch
tau_z = (-F0 + F1 + F2 - F3) * kappa      // This is Yaw

(see Slack link for details)

also notice that in this project l = sqrt(2.f) instead of 2 * sqrt(2.f) like in the exercise, because L is half the distance between rotors!

This system can be solved using Wolfram Alpha: https://www.wolframalpha.com/input/?i=solve+for+F0,F1,F2,F3++%7B%7B1,1,1,1%7D,%7B1,-1,1,-1%7D,%7B1,1,-1,-1%7D,%7B-1,1,1,-1%7D%7D+*+%7BF0,F1,F2,F3%7D+%3D+%7BA,B,C,D%7D

A lot of acknowledgement to Mike, for his explanation, not sure if I would have figured this out completely by myself.

In the method we get the collThrustCmd as a parameter which is a thrust value already unlike in the exercise where we still had to use the mass to convert c to a thrust value.

One additional difference is that we output the thrust for all four rotors instead of the angular velocities like in the exercise notebook.

### Body Rate Control

Body rate control is a simple p-Controller which outputs moments of all three axis given p,q,r (body rates). Unlike in the exercise notebook we need to use the inertia constants to convert the final value.

### Roll and Pitch Control

Again the exercise notebook can be consulted as a reference. The implementation is again a simple p-Controller with conversion from body frame to world frame. One difference compared to the exercise is that we get the collective thrust command which we have to convert to an acceleration, which is then used to convert the accelCmd into an angle.

I found it very important to constrain the angle value here using the CONSTRAIN macro and maxTiltAngle for a properly working controller!

### Altitude Control

The altitude Controller is a feed-forward PID Controller, and therefore considers past (P), present (D) and the future (I). The integration part here is not part of the exercise notebook (only uses feed-forward PD Controller). In addition, in this project we need to again constrain, in this case the acceleration value and we have to convert acceleration to thrust using the mass of the copter.

### Lateral Control

Again we implement a feed-forward PD Controller to command lateral acceleration (x and y). In this case it was important to constrain the acceleration and also the velocity command the method gets as a paremeter. To do so I implemented a little helper function which checks the vector norm in x and y and corrects the norm to a max value if necessary.

### Yaw Control

Implementation of a P Controller to control yaw angle (scalar). 

### Scenarios

After constraining some command values and tuning all the controller gains I managed to get scenario 1 to 5 passing :)

### Tracking trajectories and extra challenge

I am a little confused, it seems that the two trajectories are not the same (FigureEight and FigureEightFF), it seems that FigureEightFF has velocity information whereas FigureEight does not.

I can observe that FigureEightFF trajectory is much smoother and the error value is consistently much lower compared to FigureEight. This is because the extra velocity information gives extra (implicit) information about the trajectory. Instead of no information at all between two discrete timesteps velocity helps the controller to smoothen movements.

#### extra challenge 1

It seems that this code is already provided but commented out (```traj/MakePeriodicTrajectory.py```), the velocity between 2 timesteps is calculated using time interval and the difference in position (delta).

#### extra challenge 2

 TODO :)
