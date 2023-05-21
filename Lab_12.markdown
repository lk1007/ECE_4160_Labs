---
layout: single
title: Lab 12
permalink: /Lab_12/
author_profile: true
toc: true
sidebar:
  nav: "docs"
---

In this lab, I merged all the topics from previous lab into navigation around various waypoints in the map.

# Prototyping in simulator

Before I started doing any testing on the real map, I wanted to test an ideal trajectory creation. To do this, I thought of navigating using these steps

* Localize for current position
* Calculate required angle and distance to get to next waypoint
* Execute calculated movement
* Localize
* If Localization is too far away keep same waypoint as target, else calculate next waypoint
  
In practice, this was the code I used to execute the simulated trajectory.

```python
for waypoint in real_path:
    orig = waypoint
    waypoint = ft_to_m(waypoint[0]),ft_to_m(waypoint[1])
    while(not err(waypoint,cur_pos)):
        print("waypoint(m):",waypoint)
        print("waypoint(ft):",(orig))
        d,angle = calc_movement(cur_pos,waypoint) 
        #prev_odom, current_odom, prev_gt, current_gt = traj.execute_time_step(t)
        if(orig != (5,-3)):
            cmdr.set_vel(0,-cur_angle)
            time.sleep(1)
        cmdr.set_vel(0,angle)
        time.sleep(1)
        cmdr.set_vel(d,0)
        time.sleep(1)
        cmdr.set_vel(0,0)
        # Prediction Step
        #loc.prediction_step(current_odom, prev_odom)
        #loc.print_prediction_stats(plot_data=True)
        
        # Get Observation Data by executing a 360 degree rotation motion
        await loc.get_observation_data()
        
        # Update Step
        loc.update_step()
        bel = loc.print_update_stats(plot_data=True)
        cur_pos = bel[0],bel[1]
        cur_angle = bel[2] * math.pi/180
        print(cur_pos)
```


Here, you can see I modified loc.print_update_stats to return the belief value to use as my current position and angle.
Here is a video of the robot executing the navigation code, here some waypoints are cut off as the simulator robot is too large to move to the waypoints without touching the walls.


<video width="100%" controls>
  <source src="../videos/lab_12_sim.webm" type="video/webm">
  Your browser does not support the video tag.
</video>


# Setting up navigation
## PID

To navigate on the real robot, I needed a similar command to set_vel. To do this, I created the commands move_distance, and move_angle.

move_distance PID controls the robot a certain distance forward. Here is the code for it.

```python
def move_distance(ble,dist,dir):
    dist = dist*1000
    ble.send_command(CMD.CHANGE_PID, "|.08|0.008|.1|60|50|1.5")
    ble.send_command(CMD.DO_PID, "|3000|" + str(dist) + "|0|0|" + str(dir))
```
Here the PID values are set for TOF control and the control is executed for distance control. DO_PID will be explained later

move_angle will move the robot a certain number of degrees shown here.

```python
def move_angle(ble,angle):
    ble.send_command(CMD.CHANGE_PID, "|1.4|0.1|4|60|50|1.23")
    ble.send_command(CMD.DO_PID, "|1200|" + str(angle) + "|1|0|1")
```
Here, the same thing occurs, but with the PID variables for angular speed control and setting the pid flags for angular control.

I created the general command DO_PID that will do PID on angular or distance control. The command looks like this.

```c
curr_val = -1;
curr_distance = -1;
if (is_angle)
{
    while (!myICM.dataReady())
    {
    }
    setAGMT(&myICM);
    curr_val = yaw_g;
    desired_val = curr_val + fun_arg;
}
if (!is_angle)
{
    if (front_tof)
        vl53 = vl53_1;
    else
        vl53 = vl53_2;
    while (curr_distance == -1)
    {
        if (vl53.dataReady() && !is_angle)
        {
            getDistance(&vl53, &curr_distance);
        }
    }
    if (front_tof)
        desired_distance = curr_distance - fun_arg;
    else
        desired_distance = curr_distance + fun_arg;
}
Serial.println("entering loop");
while ((int)millis() - currTime < duration)
{
    if (myICM.dataReady() && is_angle)
    {
        setAGMT(&myICM);
        if (is_orient)
            doPID_IMU(yaw_g, desired_val);
        else
            doPID_IMU(z_g, fun_arg);
    }
    else if (vl53.dataReady() && !is_angle)
    {
        int16_t curr_distance = (int16_t)curr_val;
        int16_t prev_distance = (int16_t)curr_distance;
        getDistance(&vl53, &curr_distance);
        if (curr_distance == -1)
            curr_distance = prev_distance;
        doPID_DIST(curr_distance, desired_distance,front_tof);
    }
}
stop();
```
The process here is that the robot takes a current value from the target sensor, and then adds the desired change in angle or distance to the current value and then PID controls on that new value. 

An important part of debugging navigation was that on some areas of the map, the front TOF sensor had trouble due to the high distance. To combat this, I move the second TOF sensor to the back of the robot. The PID control process is the same, but instead of subtracting the desired distance, the TOF sensor should move to a larger distance since it is facing the opposite way of the robot.

## Navigation loop

A few things had to be modified from the simulator in the loop for the real robot. First, instead of trying to get close to the waypoint, to reduce accumulating error, the localization would simply happen once per waypoint. Next, the current angle came from the odometry model rather than the actual belief result as localization didn't provide a very accurate current angle.
Here is the code for the navigation.

```python
real_path = [(-4,-3),(-2,-1),(1,-1),(2,-3),(5,-3),(5,-2),(5,3),(0,3),(0,0)]
test_path = [(-3,-2)]
start_time = time.time()
for waypoint in real_path:
    orig = waypoint
    waypoint = ft_to_m(waypoint[0]),ft_to_m(waypoint[1])
    dir = int(input("direction"))
    #current_angle = int(input("angle"))
    print("waypoint(m):",waypoint)
    print("waypoint(ft):",(orig))
    d,angle = calc_movement(cur_pos,waypoint) 
    #prev_odom, current_odom, prev_gt, current_gt = traj.execute_time_step(t)
    move_angle(ble,(angle - current_angle)*180/math.pi)
    time.sleep(4)
    print("moving distance:",d)
    move_distance(ble,d,dir)
    time.sleep(3)
    # Prediction Step
    #loc.prediction_step(current_odom, prev_odom)
    #loc.print_prediction_stats(plot_data=True)
    
    # Get Observation Data by executing a 360 degree rotation motion
    await loc.get_observation_data()
    loc.update_step()
    bel = loc.plot_update_step_data(plot_data=True)
    cur_pos = bel[0],bel[1]
    while(not err(waypoint,cur_pos,1)):
        await loc.get_observation_data()
        loc.update_step()
        bel = loc.plot_update_step_data(plot_data=True)
        cur_pos = bel[0],bel[1]
    cur_pos = bel[0],bel[1]
    current_angle = bel[2]
    print("current position:", m_to_ft(cur_pos[0]),m_to_ft(cur_pos[1]))
    print("current angle:", m_to_ft(current_angle))
```

## Manual control

Unfortunately, the localization did not work well despite repeatedly localizing in the same spot. Because of this, the choice was made to manually tweak the robot's movements

Here is the code for manually controlling the PID values to change the robot's position with.

```c
distance = float(input("distance"))
angle = float(input("angle"))
dir = input("dir")

move_angle(ble,angle)
time.sleep(3)
move_distance(ble,ft_to_m(distance),dir)
```
Here is the video of the control.

<iframe src="https://drive.google.com/file/d/1tX1r-0XXAehmZ_5AssvleJHl_V5iDnm7/preview" width="640" height="480" allow="autoplay"></iframe>

Unfortunately, the robot hits the walls multiple times in the video. The issue is mainly with the inconsistency of the TOF sensor. Despite telling the robot to move 1 ft,
the robot would move 4ft, because the closest wall was out of range. This would probably be fixed by using the back sensor or front sensor on the closest wall when necessary thus reducing inconsistency.


