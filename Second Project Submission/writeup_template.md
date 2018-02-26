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

```
 def TF_Mat(alpha, a, d, q):
            TF = Matrix([[            cos(q),           -sin(q),           0,             a],
                        [ sin(q)*cos(alpha), cos(q)*cos(alpha), -sin(alpha), -sin(alpha)*d],
                        [ sin(q)*sin(alpha), cos(q)*sin(alpha),  cos(alpha),  cos(alpha)*d],
                        [                 0,                 0,           0,             1]])
            return TF

```
we then substitute from the DH table
```
        T0_1 = TF_Mat(alpha0, a0, d1, q1).subs(dh)
        T1_2 = TF_Mat(alpha1, a1, d2, q2).subs(dh)
        T2_3 = TF_Mat(alpha2, a2, d3, q3).subs(dh)
        T3_4 = TF_Mat(alpha3, a3, d4, q4).subs(dh)
        T4_5 = TF_Mat(alpha4, a4, d5, q5).subs(dh)
        T5_6 = TF_Mat(alpha5, a5, d6, q6).subs(dh)
        T6_7 = TF_Mat(alpha6, a6, d7, q7).subs(dh)
```
And then post-multiply to have the homogeneous transform between the base_link and the gripper_link
```
 T0_7 = simplify(T0_1 * T1_2 * T2_3 * T3_4 * T4_5 * T5_6 * T6_7)
```

#### 3. Decouple Inverse Kinematics problem into Inverse Position Kinematics and inverse Orientation Kinematics; doing so derive the equations to calculate all individual joint angles.

And here's where you can draw out and show your math for the derivation of your theta angles. 

![alt text][image2]

### Project Implementation

#### 1. Fill in the `IK_server.py` file with properly commented python code for calculating Inverse Kinematics based on previously performed Kinematic Analysis. Your code must guide the robot to successfully complete 8/10 pick and place cycles. Briefly discuss the code you implemented and your results. 


Here I'll talk about the code, what techniques I used, what worked and why, where the implementation might fail and how I might improve it if I were going to pursue this project further.  


And just for fun, another example image:
![alt text][image3]

