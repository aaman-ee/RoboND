# Project: Follow Me

[//]: # (Image References)

[image1]: ./misc/output_19_99.png
[image2]: ./misc/output_26_0.png
[image3]: ./misc/output_27_0.png
[image4]: ./misc/output_28_0.png
[image5]: ./misc/follow_me.jpg

## Network Model

Because the task of the project is to follow a target (not only what object but where) we will use a network full of convolutional layers. We will first extract important target features through encoding, and then upsample into an output image during decoding, to finally assign pixels to classes.
For a good trade-off between number of parameters and adequate image decoding, we chose a symmetrical, 5 layer model architecture with the following depth
```
Input > 64 > 128 > 256 > 128 > 64 > Output
```

For building the model we will:

  * Create encoder blocks
  
``
  layer1 = encoder_block(inputs, 64, 2)
  
  layer2 = encoder_block(layer1, 128, 2)
``
  * Create a 1x1 convolution block
  
  ``
  layer3 = conv2d_batchnorm(layer2, 256, kernel_size=1, strides=1)
  ``
  * Create decoder blocks
 
  ``
  layer4 = decoder_block(layer3, layer1, 128)
  
  layer5 = decoder_block(layer4, inputs, 64)
 ``
  

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
An epoch is a single pass through all of the training data. The number of epochs sets how many times you would like the network to see each individual image. 50 epochs seem fine since more epochs do not introduce dramatic enhancements in train and val loss.
Steps per epoch define the number of batches we should go through in one epoch. In order to utilize the full training set, it is wise to make sure ``batch_size*steps_per_epoch = number of training`` examples In our case, this is 64*100=6400, so multiple images are seen more than once by the network.
The Validation Steps are similar to steps per epoch but for the validation set. The default value provided worked fine.
Finally, workers define the number of processes we can start on the CPU/GPU. I used 120 for the AWS.

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


The model could be used to recognize and follow other shapes such as cats or dogs, with the necessary equivalent training dataset, however, since we had quite many false negatives when the target was far and considering that cats and dogs are smaller in volume than humans, an even larger set of training and validation images would be demanded. 





