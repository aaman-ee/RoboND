# Project: Follow Me

[//]: # (Image References)

[image1]: ./misc/output_19_99.png
[image2]: ./misc/output_26_0.png
[image3]: ./misc/output_27_0.png
[image4]: ./misc/output_28_0.png
[image5]: ./misc/follow_me.jpg

## Network Model

First we extract features from all 8 objects using color and surface normal features. The feature extraction was in the HSV color space for 100 different pose orientations for each object. SVM training for object recognition with linear kernel was utilized. The confusion matrix reveals adequate recognition results.



<img src="./misc/figure_1.png" width="425"/> <img src="./misc/figure_2.png" width="425"/> 

Next, and for each world, the following pipeline was used highlighting some important code parts. Note that the same parameters of the code are used for all the three worlds.

A Statistical Outlier Filtering is first applied to the point cloud of the RGB-D camera.
```
outlier_filter = pcl_datacloud.make_statistical_outlier_filter()
outlier_filter.set_mean_k(50)
x = 1.0
outlier_filter.set_std_dev_mul_thresh(x)
cloud_filtered = outlier_filter.filter() 
```
Afterwards, a Voxel Grid Downsampling and a PassThrough Filter are applied to work with less data points for time efficiency. I applied two axis filtering for elimination of the box outliers
```
vox = cloud_filtered.make_voxel_grid_filter()
LEAF_SIZE = 0.003
vox.set_leaf_size(LEAF_SIZE, LEAF_SIZE, LEAF_SIZE)
cloud_filtered = vox.filter()
passthrough = cloud_filtered.make_passthrough_filter()
filter_axis = 'z'
passthrough.set_filter_field_name(filter_axis)
axis_min = 0.6
axis_max = 1.1
passthrough.set_filter_limits(axis_min, axis_max)
cloud_filtered = passthrough.filter()
passthrough = cloud_filtered.make_passthrough_filter()  # second filter for passing out drop boxes
filter_axis = 'x'
passthrough.set_filter_field_name(filter_axis)
axis_min = 0.3
axis_max = 1.0
passthrough.set_filter_limits(axis_min, axis_max)
cloud_filtered = passthrough.filter()
```
A RANSAC Plane Segmentation is then applied for extracting (removing) table
```
seg = cloud_filtered.make_segmenter()
seg.set_model_type(pcl.SACMODEL_PLANE)
seg.set_method_type(pcl.SAC_RANSAC)
max_distance = 0.01
seg.set_distance_threshold(max_distance)
inliers, coefficients = seg.segment()
extracted_inliers = cloud_filtered.extract(inliers, negative=False)
extracted_outliers = cloud_filtered.extract(inliers, negative=True)
```
An Euclidean Clustering is applied to the remaining point cloud for extracting the point clouds for each object
```
ec.set_ClusterTolerance(0.01)
ec.set_MinClusterSize(100)
ec.set_MaxClusterSize(6000)
```
We then extract the features of the detected objects and clasify (recognize) the clusters 
```
chists = compute_color_histograms(rospcl_cluster, using_hsv=True)
normals = get_normals(rospcl_cluster)
nhists = compute_normal_histograms(normals)
feature = np.concatenate((chists, nhists))
prediction = clf.predict(scaler.transform(feature.reshape(1,-1)))
label = encoder.inverse_transform(prediction)[0]
detected_objects_labels.append(label)
```

Each detected object along with its label and its point cloud is parsed as a ROS message and `pr2_mover(detected_objects)` is finally used to compare with the pick list, and calculate the centroid of each detected object for correct grasping.

## Training

The training was performed in an AWS instance. We uploaded the training/validation/evaluation data through [WinSCP](https://winscp.net/eng/index.php) and enabled the notebook through [PuTTY](https://www.putty.org/).
After some training and trails we finalized the critical hyperparameters as follows:

```
learning_rate = 0.001
batch_size = 64
num_epochs = 50
steps_per_epoch = 100
validation_steps = 50
workers = 120
```
For the learning rate we chose `0.001` as the best trade off between fast error decrease and avoiding local minima.
The batch size is the number of training examples to include in a single iteration and was set to 64.
An epoch is a single pass through all of the training data. The number of epochs sets how many times you would like the network to see each individual image. 50 epochs seem fine since more epochs do not intorduce dramatic enhancements in train and val loss.
Steps per epoch define the number of batches we should go through in one epoch. In order to utilize the full training set, it is wise to make sure ``batch_size*steps_per_epoch = number of training`` examples In our case, this is 64*100=6400, so multiple images are seen more than once by the network.
validation_steps are similar to steps per epoch but for the validation set. The default value provided worked fine.
This is the number of processes we can start on the CPU/GPU. I used 120 for the AWS.

The training curves after 50 epochs is the following:

![alt text][image1]

with train loss: `0.0142` and validation loss: `0.0253`

We evaluated the trained network in the evaluation dataset to see how well the network can detect the hero from a distance, how often the network makes a mistake and identifies the wrong person as the target, and how well the network can identify the target while following it.
Some examples are listed below:

![alt text][image2]
![alt text][image3]
![alt text][image4]

The final grade score reached ``44.6%`` meeting the pass criteria of the project.

## Results 
For the final results, we first run the QuadSim in the Follow me mode, with Spawn people enabled.
Afterwards, the trained model along with its weights are loaded in the conda environment, and we set the Quad in following mode.
````
python follower.py --pred_viz model_weights.h5
````
Through the terminal, we are notified when the target is found, and the quad succesfully follows the target, even with other people visible. 


![alt text][image5]


 



