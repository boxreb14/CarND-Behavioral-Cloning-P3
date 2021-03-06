**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report

[image1]: ./nvidia-cnn.png "Model Visualization"

## Rubric Points
### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/432/view) individually and describe how I addressed each point in my implementation.  

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* model.py containing the script to create and train the model
* drive.py for driving the car in autonomous mode
* model.h5 containing a trained convolution neural network 
* writeup_report.md or writeup_report.pdf summarizing the results

#### 2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing 
```sh
python drive.py model.h5
```

#### 3. Submission code is usable and readable

The model.py file contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works.

### Model Architecture and Training Strategy

#### 1. An appropriate model architecture has been employed

I started with LeNet which can actually perform the job but as we are performing transfer learning & using keras, I chose to use a more powerful network and used the [Nvidia Convnet](https://arxiv.org/pdf/1604.07316v1.pdf) which is suggested in the lectures.

My model consists of a convolution neural network with a batch size of 32, and a stride of 2x2 for the first three layers and stride of 3x3 for the last 2 conv layers.The model includes RELU layers to introduce nonlinearity (code line 157-165), and the data is normalized in the model using a Keras lambda layer (code line 155). As suggested the images are also cropped in order to remove the region of image which is not relevant, hence 70 pixels from the top and 20 pixels from the bottom are removed. Finally the 4 dense layers are added to complete the model.

#### 2. Attempts to reduce overfitting in the model

The model contains dropout layers in order to reduce overfitting (model.py lines 158-166). 

The model was trained on the images generated by the generator which augments the images at random. There were four data augmentation techniques used modifying luminescence, casting shadows, shifting the image and rotating the image. The model was tested by running it through the simulator and ensuring that the vehicle could stay on the track.

#### 3. Model parameter tuning

510/509 [==============================] - 108s - loss: 0.0446 - val_loss: 0.0375
Epoch 2/10
510/509 [==============================] - 106s - loss: 0.0342 - val_loss: 0.0303
Epoch 3/10
510/509 [==============================] - 115s - loss: 0.0302 - val_loss: 0.0290
Epoch 4/10
510/509 [==============================] - 111s - loss: 0.0283 - val_loss: 0.0296
Epoch 5/10
510/509 [==============================] - 106s - loss: 0.0273 - val_loss: 0.0271
Epoch 6/10
510/509 [==============================] - 105s - loss: 0.0255 - val_loss: 0.0264
Epoch 7/10
510/509 [==============================] - 108s - loss: 0.0244 - val_loss: 0.0235
Epoch 8/10
510/509 [==============================] - 105s - loss: 0.0240 - val_loss: 0.0240
Epoch 9/10
510/509 [==============================] - 111s - loss: 0.0231 - val_loss: 0.0237
Epoch 10/10
510/509 [==============================] - 107s - loss: 0.0233 - val_loss: 0.0241

The model used an adam optimizer, so the learning rate was not tuned manually (model.py line 172). I used the batch size of 32 as the model performed better for this size, there was not much difference because the loss was already low but 20% decrease was observed in the validation loss on going higher batch size of upto 256. Also after 8th or 9th epoch usually the validation loss remained the same till about 17 epochs and goes up afterwards a bit. Also around the same number of epochs the training loss is also at its least.

#### 4. Appropriate training data

Training data was chosen to keep the vehicle driving on the road. I created the training data based on the data augmentation techniques namely modifying luminescence, casting shadows, shifting the image and rotating the image. All these augmentation techniques were included in the generator used and were chosen at random to not create huge amount of disk space. 

Also data selection was done to correct the steering angles based on the images of left and right cameras of the simulator, for that an offset of +-0.2 was applied. Also since most of the steering angles were very close to zero, the distribution was not right so to make it gaussian the steering angles of upto +-0.045 were removed. 

### Model Architecture and Training Strategy

#### 1. Solution Design Approach

The overall strategy for deriving a model architecture was to take a powerful network and use transfer learning. My first step was to use a convolution neural network model similar to the LeNet as convolutional networks are very good in taking camera images as input and giving the output of steering angles. I ended upto using the Nvidia Convnet as this network was more powerful than LeNet and was specifically designed keeping in mind that the network needs to accuratly imitate the driving behavior. And it was also suggested in the lectures.  

In order to gauge how well the model was working, I split my image and steering angle data into a training and validation set. Now as pointed out in the lectures this training data can become huge and it might be very hard to get it loaded in the server to train. 10K traffic sign images is about 30MB but 10K simulator images would consume more than 1.5GB of memory, excluding copies made during intermediate processing. This is where generator comes in, generator is a function that only return a slice of data required by the main routine and remember its state, for instance, index of images to process. The next time the generator function is called, the subsequent slice of data is returned. In this way, all the training data is not required to load into the memory.

I used the root mean square error to measure the loss and to comabat the overfitting added the droput layer after every convolutional layer with droput rate of 10%.

At the end of the process, the vehicle is able to drive autonomously around the track without leaving the road.

#### 2. Final Model Architecture

The final model architecture (model.py lines 152-172) consisted of a convolution neural network with the following layers and layer sizes.

![alt text][image1]

Layer (type)                 Output Shape              Param #   
=================================================================
cropping2d_1 (Cropping2D)    (None, 70, 320, 3)        0         
_________________________________________________________________
lambda_1 (Lambda)            (None, 70, 320, 3)        0         
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 35, 160, 24)       1824      
_________________________________________________________________
dropout_1 (Dropout)          (None, 35, 160, 24)       0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 18, 80, 36)        21636     
_________________________________________________________________
dropout_2 (Dropout)          (None, 18, 80, 36)        0         
_________________________________________________________________
conv2d_3 (Conv2D)            (None, 9, 40, 48)         43248     
_________________________________________________________________
dropout_3 (Dropout)          (None, 9, 40, 48)         0         
_________________________________________________________________
conv2d_4 (Conv2D)            (None, 3, 14, 64)         27712     
_________________________________________________________________
dropout_4 (Dropout)          (None, 3, 14, 64)         0         
_________________________________________________________________
conv2d_5 (Conv2D)            (None, 1, 5, 64)          36928     
_________________________________________________________________
dropout_5 (Dropout)          (None, 1, 5, 64)          0         
_________________________________________________________________
flatten_1 (Flatten)          (None, 320)               0         
_________________________________________________________________
dense_1 (Dense)              (None, 100)               32100     
_________________________________________________________________
dense_2 (Dense)              (None, 50)                5050      
_________________________________________________________________
dense_3 (Dense)              (None, 10)                510       
_________________________________________________________________
dense_4 (Dense)              (None, 1)                 11        
=================================================================

Initially I used a cropping layer so that the pixels from our region of interest are kept in the image. We remove the area 0f 70 pixels from the top and 20 pixels from the bottom of the image. Hence the input image is of 70 pixels high and 320 pixels wide. Now to perform the normalization a lambda layer is used. 5 Convolution layers are used then, first 3 use 5x5 kernel with 2x2 stride whereas the last 2 use 3x3 kernel with 3x3 stride. They are also followed with ReLU layers and to prevent overfitting we use droput rate of 10% after every layer. Now 4 dense layers are used to lower the hidden units. And lastly root mean square error is used as we are more concerned of the accuracy of the steering angle.

#### 3. Creation of the Training Set & Training Process

To capture good driving behavior, I checked the center, left and right images from the simulator. Left view image appears to be closer to right side of the lane, so a negative compensation needs to be added to original recorded steering angle, i.e. steering left. Whereas, a positive value will be added for the camera on the right. I was not sure of how to get the calculation right for this offset, so as suggested in the project guide, a constant offset of +-0.2 is applied.

On going through the driving log I found that the a very low interval of steering angles are wau too much in order to get the car on the middle of the road. So to make training efficient I made this steering angle distribution gaussian and removed the low steering angles of upto 0.045.

I then used data augmentation techniques like modifying luminescence, casting shadow in all four corners, shifting the image pixels horizontally or vertically and rotation of the image.

I finally randomly shuffled the data set and put 20% of the data into a validation set. 

I used this training data for training the model. The validation set helped determine if the model was over or under fitting. The ideal number of epochs was 10 as evidenced by the stagnating validation loss. I used an adam optimizer so that manually training the learning rate wasn't necessary.
