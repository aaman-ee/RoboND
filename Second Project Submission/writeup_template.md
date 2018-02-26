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
Finally, we apply a corection as proposed to consider the different orientation between URDF and DH
```
  R_corr = Matrix([[0,0,1.0,0],[0,-1.0,0,0],[1.0,0,0,0],[0,0,0,1.0]])
  T0_7_corr = (T0_7 * R_corr)
```

#### 3. Inverse Kinematics Analysis.

Since we have a special case of three last joints revolute with their joint axes intersecting at a single point, we can decouple the IK problem into Inverse Position and Inverse Orientation problems for the wrist center (WC).
The end-effector position and orientation are retrieved by the functions already provided.

We need the rotation matrix for the end-effector along with a correction:

```
 # Roll
        ROT_x = Matrix([[       1,       0,       0],
                        [       0,  cos(r), -sin(r)],
                        [       0,  sin(r),  cos(r)]])
        # Pitch
        ROT_y = Matrix([[  cos(p),       0,  sin(p)],
                        [       0,       1,       0],
                        [ -sin(p),       0,  cos(p)]])
        # Yaw
        ROT_z = Matrix([[  cos(y), -sin(y),       0],
                        [  sin(y),  cos(y),       0],
                        [       0,       0,       1]])

        ROT_EE = ROT_z * ROT_y * ROT_x
        
        ROT_corr = ROT_z.subs(y, 3.1415926535897931) * ROT_y.subs(p, -1.5707963267948966)
        ROT_EE = ROT_EE * ROT_corr
```
We then calculate the WC position

```
# Calculate WC position
    WC = EE - (0.303) * ROT_EE[:,2]
```
We have the wrist center in WC (Wx, Wy, Wz).

Using trigonometry we calculate theta1, theta2, theta3. With these angles, we can calculate R0_3 and finally R0_6
```
            R0_3 = T0_1[0:3,0:3] * T1_2[0:3,0:3] * T2_3[0:3,0:3]
            R0_3 = R0_3.evalf(subs={q1: theta1, q2: theta2, q3:theta3})
                        
            # Get rotation matrix R3_6 from (transpose of R0_3 * R_EE)
            R3_6 = R0_3.transpose() * ROT_EE
```
We finally, get the angles from 
```
theta5 = atan2(sqrt(R3_6[0,2]*R3_6[0,2] + R3_6[2,2]*R3_6[2,2]),R3_6[1,2])
theta4 = atan2(-R3_6[2,2], R3_6[0,2])
theta6 = atan2(R3_6[1,1],-R3_6[1,0])
```



### Project Implementation

#### 1. Fill in the `IK_server.py` file with properly commented python code for calculating Inverse Kinematics based on previously performed Kinematic Analysis. Your code must guide the robot to successfully complete 8/10 pick and place cycles. Briefly discuss the code you implemented and your results. 

The code is attached in the github folder, my code run quite efficiently, I had two objects lost (8/10), because I pressed quickly the button "next" during the grapsing object routine, and the gripper hasn't grabed the object.
The code needed some time for the IK calculation.


And just for fun, another example image:
![alt text][image3]

