# Project 5: Vehicle Detection

## Outline:
1. Train SVM to detect cars and non-cars based on spatial, histogram and HOG features.
2. Use sliding windows with HOG subsampling at various scales to detect vehicles in images.
3. Create cumulative heatmaps over multiple video frames to mitigate false positives.
4. Detect lanes using code from project 4.
5. Create output video using both vehicle and lane detection.

[//]: # (Image References)
[image1]: ./examples/car01.png
[image2]: ./examples/car02.png
[image3]: ./examples/car03.png
[image4]: ./examples/noncar01.png
[image5]: ./examples/noncar02.png
[image6]: ./examples/noncar03.png
[image7]: ./examples/heat01.png
[image8]: ./examples/heat02.png
[image9]: ./examples/heat03.png
[image10]: ./examples/heat04.png
[image11]: ./examples/heat05.png
[image12]: ./examples/heat06.png
[image12]: ./examples/left01.png
[image12]: ./examples/left02.png
[video1]: ./output_video_v2.mp4

### Reference: the method used in this project are adopted from Udacity's vehicle detection lessons.

### Below the [rubric points](https://review.udacity.com/#!/rubrics/513/view) are addressed individually with description of implementation.

---

## Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This is the writeup / project report.

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

HOG features are extracted in the get_hog_features() method (see section 1. Training SVM in code) using OpenCV. In the same code section the extract_features() method calls get_hog_features() along with bin_spatial() and color_hist(), and returns spatial, histogram and HOG features of a given image. This method is used to extract features from all training images.

Training images consist of car and non-car images (see examples below), all of which are 64x64 pixels. The classifier was trained on both classes to differentiate between the two.

**Car examples:**  
![car01][image1]![car02][image2]![car03][image3]
  
**Non-car examples:**   
![noncar01][image4]![noncar02][image5]![noncar03][image6]
  
#### 2. Explain how you settled on your final choice of HOG parameters.

Parameters for spatial, histogram and HOG methods were tuned to detect as many car windows as possible, without too many false positives. First I determined which color space provides the best features to differentiate cars and background imagery. YCrCb color space worked best for this purpose. Spatial parameters were tuned next and values of 32x32 were used to keep more spatial information than lower values. Lastly HOG parameters were tuned. Orient value of 9 seemed optimal as increasing beyond that value produced too many false positive detections. 8 pixels per cell and 2 cells per block provided the best detections when compared to higher or lower values of these parameters.
  
#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).
  
The code for SVM training can be found in section 1. First vehicle and non-vehicle image locations were loaded from specific directories. These locations were used to load the images and extract their features. Once extracted, all features were normalized using the StandardScaler() method. Scikit learn’s LinearSVC was used to fit a SVM for detection. 2680 vehicle and 2700 non-vehicle images were used for training and produced a test accuracy of 1.0.
  
### Sliding Window Search

#### 1 . Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

