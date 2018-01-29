**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car_hog_vis]: ./writeup/hog_vis.png
[non_car_hog_vis]: ./writeup/non_car_hog_vis.png
[heat_labels_maps]: ./writeup/heat_labels_maps.png
[all_windows]: ./writeup/all_windows.png
[output1]: ./output_images/output1.png
[output5]: ./output_images/output5.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 4th and 5th code cells of the IPython notebook.  

I used `skimage.hog()` function with parameters `orientations=9`, `pixels_per_cell=(8, 8)`, `cells_per_block=(2, 2)`, and `transform_sqrt=False` to extract HOG features from images in `YCrCb` color space.

Here is an example of HOG of a car vehicle:

![alt text][car_hog_vis]

Here is an example of HOG of a non-car vehicle:

![alt text][non_car_hog_vis]

#### 2. Explain how you settled on your final choice of HOG parameters.

The `orientations=9`, `pixels_per_cell=(8, 8)`, `cells_per_block=(2, 2)` combination gives reasonable number of parameters that are enough to capture the characteristics of an image without overloading the training and inference processes. The `transform_sqrt=False` is an interesting one. I tried to enable it, but it significantly reduces the inference accuracy, which is a bit counter-intuitive, because the purpose of that operation is to reduce the impact of variation of illumination and shadowing, which sounds good for classification problems.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using a combination of the raw pixels and HOG, which is better than using HOG alone from what I tested (code cell 6, 7, 8, 10, 11, 12). With a 9:1 train test split, it achieved `0.99` accuracy on the test set.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I searched with windows of different sizes, varying from 90 pixels to 130 pixels. And smaller windows are only applied to further portions of the image, while larger ones are applied to closer portions (code cells 17, 19).

Here are all the windows used on an image:
![alt text][all_windows]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on 5 scales using YCrCb 3-channel HOG features plus spatially binned color in the feature vector, which provided a reasonable result.  Here are some example images:

![alt text][output1]

![alt text][output5]


To improve the classification, I mainly experimented with different features on different color spaces. I started with HOG only on HLS space, and then tried gray and RGB, finally HOG plus binned color on YCrCb space produced the best result.


---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. From the positive detections I created a heatmap and then thresholded (at least 2 positives) that map to identify vehicle positions. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap. I then assumed each blob corresponded to a vehicle. I constructed bounding boxes to cover the area of each blob detected. Code is at cells 17, 18.

Here's an example result showing the heatmap and labels map for a frame of the video:

![alt text][heat_labels_maps]

And the resulting bounding boxes are drawn onto the original frame:

![alt text][output1]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

False positives from the classifier was a big problem I faced. The classifier is likely to predict positive when the HOG characteristics of a non-car image are close to what of a car one, e.g. some horizontal tree shadows on the road, some road signs. Another problem is to capture cars with different sizes, the current approach requires different sizes of dearching windows, which I imagine wouldn't scale very well.

A CNN approach might help with these issues and make the pipeline more robust.

