# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---
## Video
https://www.youtube.com/watch?v=JRocnan_-5E&feature=youtu.be

## State variables

The process model consists of six states: x- and y-position, yaw-angle, velocity, cross-track-error and orientation-error epsi.<br />
px: current location in x-axis global coordinates<br />
py: current location in y-axis global coordinates<br />
psi / yaw-angle: car orientation<br />
velocity: describes the car's velocity<br />
cross-track-error / cte: the rectangular distance between longitudinal car position / orientation and the desired trackline.<br />
orientation-error: error between desired orientation and actual orientation.<br />



## Actuators: Steering and throttle

delta: describes the steering angle value between [-1,1]. It's constrained between [-25,25] degrees.<br />
throttle: describes the throttle value between [-1,1]. -1 is for full braking, +1 for full acceleration.<br />

## Process model
All state variables future steps can be calculated by the actual state variables and actuations just as described in image 1:

![process](text/process.PNG?raw=true)

## Timestep Length and Elapsed Duration (N & dt)
Dt was set to a constant with a factor of 0.1 -> dt=0.1 . Dt is modeled as latency in this algorithm. The timestep length N was chosen as 10. Values above 10 resulted in an instable MPC. Values below 10 were too short to drive properly through curves.

## Polynomial Fitting and MPC Preprocessing

To calculate the desired waypoints correctly I added to the state variable a latency "dt" of 0.1 seconds, based on the system dynamics.

```
          px += v * cos(psi) * dt;
          py += v * sin(psi) * dt;
          psi -= v * delta / Lf * dt;
          v += acceleration * dt;
```

The given waypoints are described according to the global coordinate system. In order to estimate the desired trajectory correctly the given values have to be transformed into the local vehicle coordinate system.

```
        for (int i=0; i<ptsx.size(); i++) {

            double shift_x = ptsx[i] - px;
            double shift_y = ptsy[i] - py;

            ptsx[i] = shift_x * cos(0-psi) - shift_y * sin(0-psi);
            ptsy[i] = shift_y * cos(0-psi) + shift_x * sin(0-psi);

             }
```

To connect the waypoints I decided to fit them by a cubic function, calulated the cte and epsi:
```
          auto coeffs = polyfit(ptsx_transform, ptsy_transform, 3);
          double cte = polyeval(coeffs, 0.0); // y=0
          double epsi = psi - atan(coeffs[1] + 2 * px * coeffs[2] + 3 * coeffs[3] *pow(px,2));// x=0
```

Last but not least I updated the state variables and fed them to my MPC:

```
          const double px_act = v * dt;
          const double py_act = 0;
          const double psi_act = v * (-delta) * dt / Lf;
          const double v_act = v + acceleration * dt;
          const double cte_act = cte + v * sin(epsi) * dt;
          const double epsi_act = epsi + psi_act;




          // state in car coordniates
          Eigen::VectorXd state(6);
          //state << px_act,py_act,psi_act, v_act, cte_act, epsi_act;
          state << px_act,py_act,psi_act, v_act, cte_act, epsi_act;


          auto vars = mpc.Solve(state, coeffs);
```


## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
