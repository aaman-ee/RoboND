## Project: Kinematics Pick & Place



[//]: # (Image References)

[image1]: ./misc/image1.jpeg
[image2]: ./misc_images/misc3.png
[image3]: ./misc_images/misc2.png


### Forward Kinematic Analysis
#### 1. We first derive the DH table based on the schematic below and the `kr210.urdf.xacro` file.

We have only revolute joints in our arm so we have theta variables only.

![alt text][image1]

#### 2. Through the four individual transforms that DH convention allow us, we create individual transformation matrices for each joint. For the generalized homogeneous transform between the base_link and the gripper_link we post-multiply the individual trasformation matrices.

We make a function instead of individual matrix constructions.



#### 3. Decouple Inverse Kinematics problem into Inverse Position Kinematics and inverse Orientation Kinematics; doing so derive the equations to calculate all individual joint angles.

And here's where you can draw out and show your math for the derivation of your theta angles. 

![alt text][image2]

### Project Implementation

#### 1. Fill in the `IK_server.py` file with properly commented python code for calculating Inverse Kinematics based on previously performed Kinematic Analysis. Your code must guide the robot to successfully complete 8/10 pick and place cycles. Briefly discuss the code you implemented and your results. 


Here I'll talk about the code, what techniques I used, what worked and why, where the implementation might fail and how I might improve it if I were going to pursue this project further.  


And just for fun, another example image:
![alt text][image3]

