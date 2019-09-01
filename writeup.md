# **Highway Driving**

## Writeup
---

**Highway Driving Project**

The goals / steps of this project are the following:
* The goal of this project is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. 
* Inputs provided are car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. 
* The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, where other cars will also try to change lanes. 
* The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. 
* The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. 
* Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

---

### Reflection

### 1. Code description to achieve the goal.

1. First, I tried to move the car in a straight line using the code snippet from the lesson.
	next_x_vals.push_back(car_x+(dist_inc*i)*cos(deg2rad(car_yaw)));
  	next_y_vals.push_back(car_y+(dist_inc*i)*sin(deg2rad(car_yaw)));

2. Next, modified the code to stay in the lane instead of moving a straight line. So I used the Frenet co-ordinates here to choose a constant "d" value for the lane and increment "s" value as required. With this change the vehicle moves in a constant lane as per chosen "d".

3. I learnt that using spline library, we can smoothen the path. As suggested in the walk-through, I chose 3 points spaced 30m apart and let spline choose the "y" point for each "x" point. This smoothing out of points helped in avoiding jerk.

4. The "x" points are spaced in such a way that car goes in the desired speed. Using this formula:
	N * 0.02 * velocity = distance travelled by car.

	We know the desired velocity and we can calculate the distance (hypotenuse) by using sqrt(x2 * y2). Using these 2 parameters, we can calculate the number of points the car has to pass through and their y values are calculated using spline () operator. 

5. All these math was perfomed by tranformation of co-ordinates from global co-ord space to local co-ord space so that "x" point will be at zero reference angle with respect to the car which makes the calculations easier. I also reversed this and transformed back to global co-ords once x & y reference points are calculated using spline. 

6. To keep the speed to 50mph, I initially set the ref_vel to be 49.5, but then changed it to 0 to allow a gradual increase in velocity from 0 to 50. The velocity of the car is set to either increase or decrease at 0.224 steps (5m/sec2). This way, I could avoid going over maximum acceleration. It also helped in avoiding jerk.

7. I also used the previous path points to ensure that the car's movement is continuous. If there is no previous path points, I back-calculate the tangential points (upto to 2 points).

8. I used the car's localization & sensor fusion data to find out if there is any car ahead of us in the lane we are moving. 
For each car, get its lane information from sensor fusion. If that car is in our lane, calculate its speed of the car from vx & vy. Then I used "s" value of the car and predict the car's next position based on its speed & s. If it is within 30m, we have to flag that the our car is "too_close" to a car ahead.

9. In that case, we have to slow down immediately, but gradually and also decide to switch lane.

10. To switch lane, based on the lane our car wants to swicth, we check for other cars in that lane and use the same prediction method to check if there will be a car ahead or behind us in the 50m range. If yes, mark that lane as busy. Once all the possible lanes are checked, based on which lane is not marked, the car swictes to that lane. [Line: 147-192]

