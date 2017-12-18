## Project: Search and Sample Return
### Writeup 

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./misc/vlcsnap-2017-12-18-19h40m47s828.png
[image5]: ./misc/vlcsnap-2017-12-18-19h46m52s664.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

In the notebook I modified the `color_thresh()` function, in the `perception.py` I used three functions.
The test_mapping.mp4 is in the current folder.

```
def color_thresh(img, rgb_thresh=(160, 160, 160)):
    # Create an array of zeros same xy size as img, but single channel
    color_select = np.zeros_like(img[:, :, 0])
    # Require that each pixel be above all three threshold values in RGB
    # above_thresh will now contain a boolean array with "True"
    # where threshold was met
    above_thresh = (img[:, :, 0] > rgb_thresh[0]) \
                   & (img[:, :, 1] > rgb_thresh[1]) \
                   & (img[:, :, 2] > rgb_thresh[2])
    # Index the array of zeros with the boolean array and set to 1
    color_select[above_thresh] = 1

    # below_thresh = (img[:, :, 0] <= rgb_thresh[0]) \
    #                & (img[:, :, 1] <= rgb_thresh[1]) \
    #                & (img[:, :, 2] <= rgb_thresh[2])
    below_thresh = np.invert(above_thresh)
    color_select[below_thresh] = 0

    rock_thresh = (img[:, :, 0] < 220) \
                   & (img[:, :, 0] > 130) \
                   & (img[:, :, 1] < 180) \
                   & (img[:, :, 1] > 100) \
                   & (img[:, :, 2] < 25)
                   
    color_select[rock_thresh] = 3

    # Return the binary image
    return color_select

threshed = color_thresh(warped)
plt.imshow(threshed, cmap='gray')
```
For path trace the threshold above 160 in all the RGB channel pixels is adequate for finding the travershable path.
```
above_thresh = (img[:, :, 0] > rgb_thresh[0]) \
                   & (img[:, :, 1] > rgb_thresh[1]) \
                   & (img[:, :, 2] > rgb_thresh[2])
```
All the others are considered as obstacles with the inversion of `below_thresh = np.invert(above_thresh)`.
For rocks, after an image analysis of the calibration images I ended up to the following thresholds
```
   rock_thresh = (img[:, :, 0] < 220) \
                   & (img[:, :, 0] > 130) \
                   & (img[:, :, 1] < 180) \
                   & (img[:, :, 1] > 100) \
                   & (img[:, :, 2] < 25)
```


#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
````
def perception_step(Rover):
    # Perform perception steps to update Rover()
    # TODO: 
    # NOTE: camera image is coming to you in Rover.img
    # 1) Define source and destination points for perspective transform
    dst_size = 5
    bottom_offset = 6
    source = np.float32([[14, 140], [301, 140], [200, 96], [118, 96]])
    destination = np.float32([[Rover.img.shape[1] / 2 - dst_size, Rover.img.shape[0] - bottom_offset],
                              [Rover.img.shape[1] / 2 + dst_size, Rover.img.shape[0] - bottom_offset],
                              [Rover.img.shape[1] / 2 + dst_size, Rover.img.shape[0] - 2 * dst_size - bottom_offset],
                              [Rover.img.shape[1] / 2 - dst_size, Rover.img.shape[0] - 2 * dst_size - bottom_offset],
                              ])
    # 2) Apply perspective transform
    warped = perspect_transform(Rover.img, source, destination)
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    threshed = color_thresh(warped)
    rock_threshed = rock_thresh(warped)
    obstacle_threshed=obstacle_thresh(warped)
    # 4) Update Rover.vision_image (this will be displayed on left side of screen)
    Rover.vision_image[:,:,0] = obstacle_threshed*255
    Rover.vision_image[:,:,1] = rock_threshed*255
    Rover.vision_image[:,:,2] = threshed*255

    # 5) Convert map image pixel values to rover-centric coords
    xpix, ypix = rover_coords(threshed)
    obxpix, obypix = rover_coords(obstacle_threshed)
    rockxpix, rockypix = rover_coords(rock_threshed)
    # 6) Convert rover-centric pixel values to world coordinates
    rover_xpos = Rover.pos[0]
    rover_ypos = Rover.pos[1]
    rover_yaw = Rover.yaw
    worldmap = Rover.worldmap
    scale = 10
    x_world, y_world = pix_to_world(xpix, ypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)

    obstacle_x_world, obstacle_y_world = pix_to_world(obxpix, obypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
    rock_x_world, rock_y_world = pix_to_world(rockxpix, rockypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
    # 7) Update Rover worldmap (to be displayed on right side of screen)
    Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
    Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
    Rover.worldmap[y_world, x_world, 2] += 1

    # 8) Convert rover-centric pixel positions to polar coordinates
    rover_centric_pixel_distances, rover_centric_angles = to_polar_coords(xpix, ypix)
    # Update Rover pixel distances and angles
    Rover.nav_dists = rover_centric_pixel_distances
    Rover.nav_angles = rover_centric_angles
        
    return Rover
````
Firstly, a perpective trasnform is applied in each new Rover.img. Then for each terrain/obstacles/rock the functions `color_thresh(), rock_thresh(),obstacle_thresh()` are performed, respectively to identify terrain/obstacles/rocks.
Each color channel shows the terrain/obstacles/rocks
````
   Rover.vision_image[:,:,0] = obstacle_threshed*255
    Rover.vision_image[:,:,1] = rock_threshed*255
    Rover.vision_image[:,:,2] = threshed*255
````
Then, for all the  terrain/obstacles/rocks, we perform rover centric trasmformation and then world-centric trasnsormation by
````
  xpix, ypix = rover_coords(threshed)
    obxpix, obypix = rover_coords(obstacle_threshed)
    rockxpix, rockypix = rover_coords(rock_threshed)
````
and
````
x_world, y_world = pix_to_world(xpix, ypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)

    obstacle_x_world, obstacle_y_world = pix_to_world(obxpix, obypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
    rock_x_world, rock_y_world = pix_to_world(rockxpix, rockypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
````
with the necessary scale for upadating the worldmap.
````
 Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
    Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
    Rover.worldmap[y_world, x_world, 2] += 1
````
Finally, the Rover pixel distances and angles are updated, for use in the upcoming image.
A snapshot of the notebook output after `moviepy` is shown below:

![alt text][image4]

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

````
def perception_step(Rover):
    # Perform perception steps to update Rover()
    # TODO: 
    # NOTE: camera image is coming to you in Rover.img
    # 1) Define source and destination points for perspective transform
    dst_size = 5
    bottom_offset = 6
    source = np.float32([[14, 140], [301, 140], [200, 96], [118, 96]])
    destination = np.float32([[Rover.img.shape[1] / 2 - dst_size, Rover.img.shape[0] - bottom_offset],
                              [Rover.img.shape[1] / 2 + dst_size, Rover.img.shape[0] - bottom_offset],
                              [Rover.img.shape[1] / 2 + dst_size, Rover.img.shape[0] - 2 * dst_size - bottom_offset],
                              [Rover.img.shape[1] / 2 - dst_size, Rover.img.shape[0] - 2 * dst_size - bottom_offset],
                              ])
    # 2) Apply perspective transform
    warped = perspect_transform(Rover.img, source, destination)
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    threshed = color_thresh(warped)
    rock_threshed = rock_thresh(warped)
    obstacle_threshed=obstacle_thresh(warped)
    # 4) Update Rover.vision_image (this will be displayed on left side of screen)
    Rover.vision_image[:,:,0] = obstacle_threshed*255
    Rover.vision_image[:,:,1] = rock_threshed*255
    Rover.vision_image[:,:,2] = threshed*255

    # 5) Convert map image pixel values to rover-centric coords
    xpix, ypix = rover_coords(threshed)
    obxpix, obypix = rover_coords(obstacle_threshed)
    rockxpix, rockypix = rover_coords(rock_threshed)
    # 6) Convert rover-centric pixel values to world coordinates
    rover_xpos = Rover.pos[0]
    rover_ypos = Rover.pos[1]
    rover_yaw = Rover.yaw
    worldmap = Rover.worldmap
    scale = 10
    x_world, y_world = pix_to_world(xpix, ypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)

    obstacle_x_world, obstacle_y_world = pix_to_world(obxpix, obypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
    rock_x_world, rock_y_world = pix_to_world(rockxpix, rockypix, rover_xpos,
                                rover_ypos, rover_yaw,
                                worldmap.shape[0], scale)
    # 7) Update Rover worldmap (to be displayed on right side of screen)
    Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
    Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
    Rover.worldmap[y_world, x_world, 2] += 1

    # 8) Convert rover-centric pixel positions to polar coordinates
    rover_centric_pixel_distances, rover_centric_angles = to_polar_coords(xpix, ypix)
    # Update Rover pixel distances and angles
    Rover.nav_dists = rover_centric_pixel_distances
    Rover.nav_angles = rover_centric_angles
    
     
    return Rover
````
    
Here I used three functions `color_thresh`, `obstacle_thresh` and `rock_thresh` for identifying navigable terrain/obstacles/rock samples, respectively. Their warped output was used to update the `Rover.vision_image` in the three different color channels. All terrain/obstacles/rock pixels were converted to rover centric and then to world centric coordinates for updating the worldmap.
Finally the `Rover.nav_dists` and `Rover.nav_angles` were updated for use in the new upcoming Rover image.

````
def decision_step(Rover):

    # Implement conditionals to decide what to do given perception data
    # Here you're all set up with some basic functionality but you'll need to
    # improve on this decision tree to do a good job of navigating autonomously!
    # Check if we have been stuck

    #int(Rover.pos[0])
    #int(Rover.pos[1])
    Rover.q.append([round(Rover.pos[0],1),round(Rover.pos[1],1),int(Rover.yaw)])
    print(Rover.q)
    #print(Rover.q.count([round(Rover.pos[0],1),round(Rover.pos[1],1),int(Rover.yaw)]))
    if Rover.q.count([round(Rover.pos[0],1),round(Rover.pos[1],1),int(Rover.yaw)]) >= 15 and Rover.laststeer == 'right':
        Rover.throttle = 0
        Rover.brake = 0
        Rover.steer = 15
        Rover.mode = 'stacked'
        print('Stached1')

    if Rover.q.count([round(Rover.pos[0], 1), round(Rover.pos[1], 1), int(Rover.yaw)]) >= 15 and Rover.laststeer == 'left':
        Rover.throttle = 0
        Rover.brake = 0
        Rover.steer = -15
        Rover.mode = 'stacked'
        print('Stacked2')

    if Rover.q.count([round(Rover.pos[0], 1), round(Rover.pos[1], 1), int(Rover.yaw)]) < 15:
        Rover.mode = 'forward'



    # Example:
    # Check if we have vision data to make decisions with
    if Rover.nav_angles is not None:
        # Check for Rover.mode status
        if Rover.mode == 'forward': 
            # Check the extent of navigable terrain
            if len(Rover.nav_angles) >= Rover.stop_forward:  
                # If mode is forward, navigable terrain looks good 
                # and velocity is below max, then throttle 
                if Rover.vel < Rover.max_vel:
                    # Set throttle value to throttle setting
                    Rover.throttle = Rover.throttle_set
                else: # Else coast
                    Rover.throttle = 0
                Rover.brake = 0
                # Set steering to average angle clipped to the range +/- 15
                Rover.steer = np.clip(np.mean(Rover.nav_angles * 180/np.pi), -15, 15)
            # If there's a lack of navigable terrain pixels then go to 'stop' mode
            elif len(Rover.nav_angles) < Rover.stop_forward:
                    # Set mode to "stop" and hit the brakes!
                    Rover.throttle = -0.1
                    # Set brake to stored brake value
                    Rover.brake = Rover.brake_set
                    #Rover.throttle = 0
                    #Rover.steer = 0
                    Rover.mode = 'stop'

        # If we're already in "stop" mode then make different decisions
        elif Rover.mode == 'stop':
            # If we're in stop mode but still moving keep braking
            if Rover.vel > 0.2:
                Rover.throttle = 0
                Rover.brake = Rover.brake_set
                Rover.steer = 0
            # If we're not moving (vel < 0.2) then do something else
            elif Rover.vel <= 0.2:
                #Rover.brake = 0
                # Now we're stopped and we have vision data to see if there's a path forward
                if len(Rover.nav_angles) < Rover.go_forward:
                    #Rover.throttle = 0
                    # Release the brake to allow turning
                    #Rover.brake = 0
                    #Rover.throttle = -0.1
                    print(np.mean(Rover.nav_angles * 180/np.pi))
                    # Turn range is +/- 15 degrees, when stopped the next line will induce 4-wheel turning
                    if np.mean(Rover.nav_angles * 180/np.pi)>=-45 or  np.mean(Rover.nav_angles * 180/np.pi)=='nan':
                        #Rover.throttle = 0
                        #Rover.brake = Rover.brake_set
                        Rover.brake = 0
                        Rover.steer = 15
                        Rover.laststeer = 'left'

                        #Rover.throttle = -Rover.throttle_set
                        #Rover.steer = 15 # Could be more clever here about which way to turn
                    elif np.mean(Rover.nav_angles * 180/np.pi)<-45:
                         #Rover.throttle = -Rover.throttle_set
                         #Rover.throttle = 0
                         #Rover.brake = Rover.brake_set
                        Rover.brake = 0
                        Rover.steer = -15
                        Rover.laststeer = 'right'
                    else:
                        if Rover.laststeer == 'right':
                            Rover.steer = -15
                        else:
                            Rover.steer = 15

                # If we're stopped but see sufficient navigable terrain in front then go!
                if len(Rover.nav_angles) >= Rover.go_forward and Rover.mode !='stacked':
                    # Set throttle back to stored value
                    Rover.throttle = Rover.throttle_set
                    # Release the brake
                    Rover.brake = 0
                    # Set steer to mean angle
                    Rover.steer = np.clip(np.mean(Rover.nav_angles * 180/np.pi), -15, 15)
                    Rover.mode = 'forward'
   
        
    # If in a state where want to pickup a rock send pickup command
    if Rover.near_sample and Rover.vel == 0 and not Rover.picking_up:
        Rover.send_pickup = True
    
    return Rover
 ````
In the `decision_step()` I introduced one more mode for the Rover 'Stacked' where this mode is initiated when all x,y,yaw values remain the same for a small period of time. This is achived through `self.q = _collections.deque(maxlen=20)` where I store the last 20 X,y,yaw values and check if more than 15 (rounded) values are the same in the que. When Stacked flag is initiated, a turn maneuver is applied.
When the Rover is stopped and there is enough vision forward data, the turn is based upon the `np.mean(Rover.nav_angles * 180/np.pi)` citeria for turnting left or right. 
    
#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

The video output file can be found in the following Dropbox Link:
https://www.dropbox.com/s/7jgqjbeu3tdwjpt/Rover%20Simulator%2014-Dec-17%205_49_45%20PM.mp4?dl=0

Video and Resolution Settings:  
64-Bit Rover Simulator  
Screen Resolution: 1024x768  
Graphics Quality: Good  
Windowed: Yes  
FPS: 27  
Pick up Rock Routine Implemented: No  

The Rover can easily avoid obstacles, map efficiently the world, and has the intelligence to understand if it has been stacked.
Rocks are identified and located in the worldmap efficiently and the rover can espace from dead-ends. The only drawback is that it might not map the entire world and miss some paths. This could be resolved if we apply a labyrinth concept of always moving with obstacles from one side. This means that the rover should move forward always and close to the rocks in its right (or left) side.  

A snapshot of the video output after `drive_rover.py` is shown below:

![alt text][image5]





