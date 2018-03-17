# CarND-Path-Planning-Project

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

### Process
I broke the problem up into 3 main sections.
1. Get the car to stay in its lane and turn smoothly
2. Get the car to accelerate and decelerate in that lane at reasonable levels, depending on the speed limit and other cars
3. Have the car change lanes when it can and should

## Keep in lane
Achieving a simple version of lane-keeping is trivial assuming you have the ability to convert from the car's Frenet coordinates to XY coordinates.  You just need pick points in the path that have a consistent `d` and a consistently increasing `s`.  Since `d` is 0 at the double-yellow line and each lane has a length of 4, setting `d` to 2 should keep it centered in the left lane. 

The issue with this approach is the choppiness of the path.  The car moving directly from these generated points makes no attempt to smooth out curves, so the acceleration and jerk from turns the will exceed comfortable levels.  Two common approaches to fixing this are generating smooth paths by fitting your points to a polynomial or a spline, which is essentially a piecewise polynomial function.  In my example I fit the points into a spline and used an externl spline library to help.

We feed a spline object points, in Frenet, such as `car_s+30,2 +4*lane`, `car_s+60,2 +4*lane`, and `car_s+90,2 +4*lane`.  We can also calculate the distance apart to place our path points so that it will drive at a target velocity, which in our case will be 49.5 MPH unless there are cars in front of us.

## Proper speed and accerlation

The approach so far will turn smoothly but it still accelerates too quickly at the start and ignores other cars in its lane.  To fix this, I first started the reference velocity at 0.  This reference velocity, `ref_vel` is used in the calculation for figuring out how far forward to place the s coordinates that we feed the spline so we have the ability to use it to tune speed and acceleration.  Then, to get the accerlation working I do this (simplified):

    if (approaching_a_car)
    {
    ref_vel -=.224;
    }
    else if (ref_vel<49.5)
    {
    ref_vel +=.224;
    }
    
The value `.224` is used because it results in accerlations of only about 5 mps^2, far less than the threshold.  The variable `approacing_a_car` of course requires the car's sensor information, and it comes from this (simplified):

    if(check_car_d < (2+4*lane+2) && check_car_d > (2+4*lane-2))
    {
      if((check_car_s > car_s) && ((check_car_s-car_s) <35))
          {
          approaching_a_car = true;
          }
    }
The first if statement is checking if the 'check car' is in our lane, and the second is checking if it's in front of us, and within 35 meters.  Done together, this tells us when when we have to slow down for a car.


## Change lanes

The first question here is to pick a general strategy for when to change lanes.  The main lane-change logic I went with followed this pseudo code.
```
if (approaching_a_car){
  if (can_change_left && didnt_just_change_lanes){
    change_left()
  }
  else if (can_change_right && didnt_just_change_lanes){
    change_right()
  } else {
    slow_down()
  }
}
```
While there are certainly better ways to do this that would use cost functions that take into account all of the cars the sensors see, this was a simple approach that prevented huge amounts of lane changes.  Without the `didnt_just_change_lanes` flag, the car sometimes attempted to change two lanes at a time and then zip back, which broke the acceleration limits and was generally just too aggressive.  The variable started as true and was set to false every time it changed lanes, and back to true when it slowed down.   The mechanics of changing the lane was easy since the lane variable was used in the points fed to the spline object in the first part– incrementing or decrementing the lane variable automatically generated paths smoothed out by the spline.
