---
layout: single
title: Lab 11
permalink: /Lab_11/
author_profile: true
toc: true
sidebar:
  nav: "docs"
---

In this lab, I merged the simulation code from lab 10 with input from the robot to do localization using the Bayes Filter

# Getting observation data

To get observation data from the robot, the data had to come in 20 degree increments. Since I was doing angular speed control, the way to get these 20 degree measurements was to estimate the change in angle based off the time between measurements and the angular speed measurements themselves. Once my estimated change in angle was more thant 20 degrees, I would take a distance measurement. Here is the code that does this.

```c++
while (cnt_d < 18)
        {
            int time = (int)millis();

            if (myICM.dataReady())
            {

                setAGMT(&myICM);

                sum += (z_g * ((int)millis() - samp_time)) / 1000.0 * 3.14159;
                if (sum > 20)
                {
                    Serial.println(cnt_d);
                    sum -= 20;
                    timeDBuff[cnt_d] = time - currTime;
                    getDistance(&vl53_1, &(dist1Buff[cnt_d]));
                    // getDistance(&vl53_2, &(dist2Buff[cnt_d]));
                    cnt_d++;
                }
                doPID_IMU(z_g, (int16_t)fun_arg, motorBuff, motorBuff, cnt_i);
                samp_time = (int)millis();
            }
        }
```

# Localization

## get_pose
Since we were testing our poses in known parts of the room, the result of get_pose was simply the point ((5,3),(0,3)...) converted to meters

## perform_observation_loop

Since I used a notification handler, I needed to use asynchronous io in my method. To do this, I followed the lab handout resulting in this code.

```python
import asyncio
data = get_data()
self.ble.send_command(CMD.GET_OBS, "|4000|" + str(90))
await asyncio.sleep(11)
n,p = (np.array(data["dist1"])[np.newaxis].T/1000 ),np.array([])
print("n:",n,"p:",p)
return n,p
```

As seen, my sensor data collection command is called GET_OBS with a collection duration and angular speed specification. I then convert this to the correct numpy array as well as to meters.

## Results

From running my localization in each spot, I got these results,

### (5,3)
<img src="../images/(5, 3).png" alt="Italian Trulli" width="100%">

### (0,3)
<img src="../images/(0, 3).png" alt="Italian Trulli" width="100%">

### (5,-3)
<img src="../images/(5, -3).png" alt="Italian Trulli" width="100%">

### (-3,-2)
<img src="../images/(-3, -2).png" alt="Italian Trulli" width="100%">

From these results, (-3,-2) lined up perfectly with the known position. This is because (-3,-2) has the most walls and the most unique layout out of the 4 locations, thus there were plenty of unique distance values to pinpoint a location. On the other hand the rest of the points were off by about a foot or two. This is because there are not as many walls or the walls were too far away to tell where exactly the robot was. This can be seen strongly in (0,3) and (-5,-3) as their wall layout is very similar so results like this would be seen where the robot thought it was in the similarly layed out location.

<img src="../images/(5, -3)bad.png" alt="Italian Trulli" width="100%">

Hopefully if this inconsistency is rare, then my localization method should be close enough to have the robot map through the arena in lab 12.
