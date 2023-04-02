---
layout: page
title: Lab 7
permalink: /Lab_7/
---
In this lab, PID control was implemented on the artemis in order to simulate the simplest control using one of the sensors on the robot.

### Estimate Drag and Momentum

To estimate drag and momentum, as said in class,

$$ d = \frac{u}{\text{steady state derivative}} $$

and

$$ m = \frac{d t_{0.9}}{ln(1s-.9s)} $$

where d and m are the drag and mass respectively.

To find this, I used a constant step response towards a wall at 
a pwm value of 200 since this was the maximum pwm value from my pid trials. Doing a step response resulted in this time of flight data.

<img src="../images/step.png" alt="Italian Trulli" width="100%">

As seen from the graph, between the two marked points, the speed reaches steady state at a derivative of about 22613mm/s

Looking at the graph, the 90% rise time was about .8s.
Therefore, 

$$ d = \frac{u}{\text{22613mm/s}} \approx 0.000383$$

$$ m = \frac{0.000383 * .8s}{ln(1s-.9s)} \approx 0.0001329 $$

Now, the A and B matrices are given by

$$ A = \begin{bmatrix} 0 & 1 \\  0 & -\frac{d}{m} \end{bmatrix} = \begin{bmatrix} 0 & 1 \\  0 & -2.88 \end{bmatrix}$$

$$ B = \begin{bmatrix} 0 \\ \frac{1}{m} \end{bmatrix} = \begin{bmatrix} 0 \\ -60175 \end{bmatrix} $$

$$ C = \begin{bmatrix} -1 & 0 \end{bmatrix} $$

### Initialize KF

For discretizing the previously computed matrices, I needed a chosen sample rate
to predict filter values at, I chose a sample rate of 15ms, so 
To discretize these matrices, the formula is 

$$ A_d = (I + dt*A) = \begin{bmatrix} 1 & 0.015 \\  0 & 0.957 \end{bmatrix} $$

$$ B_d = dt*B = \begin{bmatrix} 0 \\  -902.7 \end{bmatrix} $$

Next, I needed to 

Estimate the noise variables, sigma_u and sigma_z.

$$ \Sigma_z = \begin{bmatrix} \sigma_z^2 \end{bmatrix} $$

$$ \Sigma_u = \begin{bmatrix} \sigma_x^2 & 0\\  0 & \sigma_v^2 \end{bmatrix} $$