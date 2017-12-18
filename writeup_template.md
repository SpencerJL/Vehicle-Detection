## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/hog_feature.png
[image3]: ./output_images/search_window1.png
[image4]: ./output_images/search_window2.png
[image5]: ./output_images/search_window3.png
[image6]: ./output_images/search_window4.png
[image7]: ./output_images/search_window5.png
[image8]: ./output_images/heat_map.png
[image9]: ./output_images/scipy.png
[image10]: ./output_images/after_hot_map.png
[image11]: ./output_images/example_image.png
[video1]: ./project_video_output_with_lane_line.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

The code for HOG features is contained in the section of 'Extract Features from Image'. I then explored different color spaces and different slimage.hog() parameters (orientations, pixels_per_cell, and cells_per_block).I tried different parameters to compare the process time and combined with the features extracting been described later to get the best combination. 

Here is an example using the YUV color space and HOG parameters of orientations = 8,pixels_per_cell = (8,8), and cells_per_block = (2,2):

![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and find out the best combination of parameters are orientations = 11,pixels_per_cell = (8,8), and cells_per_block = (2,2) with the spatial size = (32,32). Using a large value of orientation that 11 didn't increase the resutlts accuracy and only increased the feature vector. Meanwhile, using a lower value of pixels_per_cell can increase the accuracy.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using images converted to all channels with the parameter described above and using the spatial intensity, channel intensity histogram features and HOG features and was able to achieve a highest test accuracy of 99.268%. The code was shown in the section of 'Train the Classifier'.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The sliding window serach was applied in the section of 'Detect vehicle with classifier in one image'. The code 'find_car' was adapted from lectures. The method combined features extraction with a sliding window search. In order to save processing time, the features extracted were subsampled according to the size of window and then been delievered into the classifier instead of extracted the features on each window individually. The prediction of features was performed for the windows and returned a list of objects corresponding to the rectangle windows. The results of search window and find_car are shown below:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]

I tried multiple scales of window and found smaller scales delievered too many false positives. At first, the window overlap was same in x and y directions, then I found when the x direction has a low overlap value but y direction has a high overlap value, there are more true positive detections. Meanwhile, only a section of vertical range of the image was considered with the part of high possibility to have car for each windown size to reduce the false positives.

The figure below presents the rectangles returned by find_cars drawn onto test image in the final implementation. There are several positive predictions on the car sections. It should be noted that there is a positive prediction of a incoming on the opposite lane.

![alt text][image6]

As described above, the positive detections were accompanished. However, in order to remove the false positives, a combined heatmap and threshold was used. The 'add_heat' function was used to detect the locations of more overlapping rectangles, which were assigned with more heat while the other sections were all black. The following image is the heatmap with a threshold was applied to and the threshold value was 1:

![alt text][image8]

The 'scipy.ndimage.measurements.label()' function collects spatially contiguous areas of the heatmap and assigns each a label as shown below:

![alt text][image9]

After these process, the final detection area was set:

![alt text][image10]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image11]

As shown in figure, the final implementation performs very well as identifying the near-field vehicles in each of the images with no false positives. However, the process at the first was not very well even I got 98.3% accuracy. I used all channels of YUV, lower pixel_per_cell and more orientations to increase the accuracy to 99.268%. Meanwhile, changing the pixels_per_cell parameter from 16 to 8 decrease the execution speed. Moreover, trying to increase the heatmap threshold from 1 to 2 can improve the accuracy of detection. However, higher threshold value might underestimate the size of the vehicle which cause no detection in image when the threshold was higher than 2 or 3 depends on different images. The other optimizations were talked above such as the overlap value and detection part in vertical direction.
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The code to process video is contained in the section of 'Pipeline for Video' and is identical to the code for processing a single image described above. The different between single image and video is the last 15 frames of video were stored by the class 'Vehicle_Detect' with the 'prev_rects' parameter. The 15 frames stored in the class were combined and added to the heatmap with the threshold value of 1 + len(det.prev_rects)/2 which performed the best results.

![alt text][video1]
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems I have met were mainly on the true and false positives. Scanning windows using classifier with different parameters delievered different results each time. Even for the same parameters in the process of image the positives presented were different sometimes. The positives searching windows appeared and disappeared on the opposite direction cars. 
Moreover,as the Python 3.6 was really unstable, the system was crushed sometimes with error 6, handle probelm, I found the searching windows presented different results in the video with or without the lane lines drawing even with the same parameters every time. 
Or for the same parameters with same video just for the car detection, no lane line detection added, the results are still different each time include the classifier accuracy varied from 98% to 99% and positive searching windows. By integrating the detections from the last 15 frames, the misclassification and false detections were reduced but still exist. 
Another problem is the vehicle can not be detected when the position of car was moved a lot from one frame to another. This can be fixed with higher overlapping window. 
This pipeline is probably fail in the case of incoming car, also the distant car as the smaller window scales tended to produce more false positives. The best way to remove the false is using high accuracy classifier and high overlap in the search window. However, this is time consuming. Maybe this can be solved with convolutional neural network and transfer learning. Meanwhile, it would be very nice to detect car speed to identify the incoming cars and distant cars.

