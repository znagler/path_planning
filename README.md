# CarND-Path-Planning-Project

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

### Process
I broke the problem up into 3 main sections.
1. Get the car to stay in its lane and turn smoothly
2. Get the car to accelerate and decelerate in that lane at reasonable levels, depending on the speed limit and other cars
3. Have the car change lanes when it can and should

## Keep in lane
Th

## Proper speed and accerlation

## Change lanes

The main lane-change logic I went with followed this pseudo code.
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
