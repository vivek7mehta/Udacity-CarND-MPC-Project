## CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program


---

### Implementation reflection

---

#### The Model

* Model consists of six state variables:
    * X-coordinate of car
    * Y-coordinate of car
    * Orientation of car
    * Velocity of the car
    * Cross-track error
    * Orientation error


* Model has two actuators:
    * Steering angle
    * Acceleration


* Update of state variables is done using kinematic vehicle model given in classroom:
  
    * x<sub>t+1</sub> = x<sub>t</sub> + v<sub>t</sub> &bull; cos(&Psi;<sub>t</sub>) &bull; dt
    * y<sub>t+1</sub> = y<sub>t</sub> + v<sub>t</sub> &bull; sin(&Psi;<sub>t</sub>) &bull; dt
    * &Psi;<sub>t+1</sub> = &Psi;<sub>t</sub> + <sup>v<sub>t</sub></sup>&frasl;<sub>L<sub></sub></sub> &delta; &bull; dt
    * v<sub>t+1</sub> = v<sub>t</sub> + a<sub>t</sub> &bull; dt
    * cte<sub>t+1</sub> = y<sub>t</sub> - f(x<sub>t</sub>) + v &bull; sin(&Psi;<sub>t</sub>) &bull; dt
    * e&Psi;<sub>t+1</sub> = &Psi;<sub>t</sub> - arctan(f '(x<sub>t</sub>)) + <sup>v<sub>t</sub></sup>&frasl;<sub>L<sub></sub></sub> &delta;<sub>t</sub> &bull; dt



* Apart from above mentioned variables, model has a cost function. Hence this problem becomes a optimization problem. Here task of MPC would be to find values for these variable so that cost function becomes minimum.


* I have added following costs in the cost function:
    * `cross-track error` &#8658; this will ensure that vehicle is following desired trajectory.
    * `orientation error`  &#8658; this will ensure that vehicle is aligned in the direction of tangent of trajectory.
    * `(actual velocity - desired velocity)` &#8658; this will ensure that vehicle matches desired velocity.
    * `steering angle` &#8658; this will ensure that steering value stays in within safe range.
    * `Throttle` &#8658; this will ensure that throttle does not go too much high and drive vehicle off the track
    * `difference of steering angle of consecutive time steps` &#8658; this will restrict car from taking sharp turns.
    * `difference of acceleration of consecutive time steps` &#8658; so that there is not sudden brake or throttle. 


* I have added weights to each cost component based on our desired behaviour. For example it will be fine if car moves slight left or right of the desired trajectory, but it will not be fine if it changes steering suddenly (otherwise car will go off the track), so we can multiply cost of `difference of steering angle of consecutive time steps` with some number greater than  1. Same way I have given weights to all the costs.

#### Timestep Length and Elapsed Duration (N & dt)
* First I decided value of T (N*dt) to 1 second which is sufficient enough but not too large because environment is changing frequently and we will only apply first actuators after that we are going to predict trajectory again. Then I tried to reduce value of dt (this will result in sufficient frequent actuations, which ensures good approximation of desired trajectory).
* First I tried with N=25 and dt=0.05 which makes car to wobble more. Then I tried N=5 and dt=0.2 which gave better approximation then previous values but not so good when I increase speed.
* Finally I came up with these values N=10 and dt=0.1. Which worked fine on my desired speed.

#### Polynomial Fitting and MPC Preprocessing
* Before fitting polynomial preprocessing has been done for waypoints transformation and to address latency.
* Since `ptsx` and `ptsy` is giving trajectory coordinates in global map coordination, and simulator takes coordinates with respect to car coordinates for visualization, this geometric transformation is done before polynomial fitting. Also after this transformation car coordinates becomes (0,0), hence it becomes mathematically easier to solve optimization problem.
*  Once this transformation is done state variables are changed to address simulator latency which is described in next section.
*  After this preprocessing coordinates are fitted into third order polynomial using `polyfit` function.

#### Model Predictive Control with Latency
* Simulator has latency of 100ms. One way to handle this latency is that we find state of the vehicle after duration of latency. Now this becomes our new state.
* I have taken 100ms as latency and used kinematic vehicle model equations to find state of the car after 100ms now this new state becomes my current state and I pass this state to my MPC. This is how I have handled latency of 100ms.