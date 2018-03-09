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
[image13]: ./examples/left01.png
[image14]: ./examples/left02.png
[video1]: ./output_video_v2.mp4

### The final output video can be found [here](./output_video_v2.mp4)

#### Reference: the method used in this project are adopted from Udacity's vehicle detection lessons.

---
  
### Histogram of Oriented Gradients (HOG)
  
HOG features are extracted in the get_hog_features() method (see section 1. Training SVM in code) using OpenCV. In the same code section the extract_features() method calls get_hog_features() along with bin_spatial() and color_hist(), and returns spatial, histogram and HOG features of a given image. This method is used to extract features from all training images.
  
Training images consist of car and non-car images (see examples below), all of which are 64x64 pixels. The classifier was trained on both classes to differentiate between the two.
  
**Car examples:**  
![car01][image1]![car02][image2]![car03][image3]
  
**Non-car examples:**   
![noncar01][image4]![noncar02][image5]![noncar03][image6]
  
Parameters for spatial, histogram and HOG methods were tuned to detect as many car windows as possible, without too many false positives. First I determined which color space provides the best features to differentiate cars and background imagery. YCrCb color space worked best for this purpose. Spatial parameters were tuned next and values of 32x32 were used to keep more spatial information than lower values. Lastly HOG parameters were tuned. Orient value of 9 seemed optimal as increasing beyond that value produced too many false positive detections. 8 pixels per cell and 2 cells per block provided the best detections when compared to higher or lower values of these parameters.
  
The code for SVM training can be found in section 1. First vehicle and non-vehicle image locations were loaded from specific directories. These locations were used to load the images and extract their features. Once extracted, all features were normalized using the StandardScaler() method. Scikit learn’s LinearSVC was used to fit a SVM for detection. 2680 vehicle and 2700 non-vehicle images were used for training and produced a test accuracy of 1.0. Training on more images did not seem to improve results.
  
 ---
  
### Sliding Window Search
  
Section 2. Vehicle Detection contains the code for performing the sliding window search. The find_cars() method extracts spatial, histogram and HOG features from the given image and uses the SVM to produce detections. This method returns all windows (bounding boxes) with detections. I experimented with various scale values (from 0.5 to 2.5). Lower values produced more false positives, whereas larger values did not detect cars further down the road. For my final implementation I used 4 scale values (0.9, 1.1, 1.4 and 1.7) and added all their heat maps together.  
  
With the parameters and scale values described above the classifier performed well. I tested each scale value separately on the video to ensure there were few false positives (if any). With all the scale values combined the resulting heat maps showed distinct hotspots. There were a few false positives; however, their heat maps were much cooler than actual car detections, as shown below:
  
![heat01][image7]  
![heat02][image8]  
![heat03][image9]  
![heat04][image10]  
![heat05][image11]  
![heat06][image12]  
  
---

### Tuning
  
I used scale values which minimized false positives and combined all their detected windows together. This resulted in few false positives which were filtered out using heat map thresholding (see code section 2. Vehicle Detection). Furthermore, I used a circular buffer (deque) to store heat values from 25 frames, with heat threshold of 32, to remove false positives. This worked well, however there are still some minor false positive detections in the video that can be removed by further tuning. Unfortunately my data pipeline takes a long time (over an hour) to process the whole video, which makes it difficult to fine-tune. 
  
---

### Discussion
  
The main issue I ran into was the trade-off between detecting vehicles and eliminating false positives. Certain parameter values resulted in the vehicles not being detected (such as higher scale values, and more pixels per cell for HOG features). Lower scale and pixel per cell values produced more detections, but also increased false positives. In my current implementation I've balanced the two as best as possible.

The SVM had a hard time detecting the white car. This could be because most of the car images it was trained on are darker. To increase this detection, I retrained the SVM with more images of the white car from the video itself (see image below). This is not the best solution since its training on a particular case and might not generalize well.

![left01][image13]  

Another issue worth noting is that currently my implementation takes over an hour to process a video less than 1 minute. This clearly cannot work in real-time, as it would need to in an actual self-driving car. An implementation in C/C++ using hardware support would greatly reduce the processing time.
