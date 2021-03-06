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
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook   file `vehicle_detecion.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images from all forder in dataset in 
`model_trannibng.ipynb`. then I added all image into two arrays cars[] and notcars[].  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

[image1]: ./output_images/car_not_car1.jpg
[image2]: ./output_images/car_not_car2.jpg
[image3]: ./output_images/car_not_car3.jpg
[image4]: ./output_images/car_not_car4.jpg

I write a sparate class for features extraction. `feature_extraction.py` is a calss where get_hog_features() function is defined.
I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

[image1]: ./output_images/hog_image1.png
[image2]: ./output_images/hog_image2.png
[image3]: ./output_images/hog_image3.png
[image4]: ./output_images/hog_image4.png

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=11`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:



#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters but below combination give me best result

color_space='YCrCb'
spatial_size=(16, 16)
hist_bins=16
orient=11
pix_per_cell=8
cell_per_block=2
hog_channel='ALL'
spatial_feat=True
hist_feat=True
hog_feat=True

Test Accuracy of SVC =  0.987

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using LinearSVC in `model_tranning.ipynb` file. The extracted features(Feature vector length: 8460) where fed to LinearSVC model of sklearn with default setting of square-hinged loss function and l2 normalization. The trained model had accuracy of 98.7% on test dataset. 
The trained model along with the parameters used for training were written to a pickle file to be further used by vehicle detection pipeline.


### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?


A single function, `find_cars` in `features_extraction.py`, is used to extract features using hog sub-sampling and make predictions. The hog sub-sampling helps to reduce calculation time for finding HOG features and thus provided higher throughput rate. A sample output from the same is shown below.

[image1]: ./output_images/feature_extraction.png

The multi-scale window approach prevents calculation of feature vectors for the complete image and thus helps in speeding up the process. The following scales were emperically decided each having a overlap of `75%` (decided by `cells_per_step` which is set as `2`). code is written in `vehicle_detection.ipynb` file's `VehicleDetector` class and function name is `draw_multi_scale_windows()`. I used different parameter and scale for multi-scale window approach.

(ystart, ystop, scale) in [(380, 480, 1), (400, 600, 1.5), (500, 700, 2.5)]

[image1]: ./output_images/multi_scale_image.png

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

[image1]: ./output_images/pipeline_image1.png
[image2]: ./output_images/pipeline_image2.png
[image3]: ./output_images/pipeline_image3.png

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:

### Here the resulting bounding boxes are drawn onto the last frame in the series:

[image1]: ./output_images/heatmap_bbox_image1.png
[image2]: ./output_images/heatmap_bbox_image2.png
[image3]: ./output_images/heatmap_bbox_image3.png
[image4]: ./output_images/heatmap_bbox_image4.png
[image4]: ./output_images/heatmap_bbox_image5.png
[image4]: ./output_images/heatmap_bbox_image6.png


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In shaow area ther are some false detection. After applying threshold still some problem. At some place there are fluckring of bounding box. 

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

For better car detection i took 10 frame avarage to draw bounding box.The search was optimized by processing complete frames only once every 10 frames, and by having a restricted search in the remaining frames.

