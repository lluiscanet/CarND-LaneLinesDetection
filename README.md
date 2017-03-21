## **Finding Lane Lines on the Road** 

Lluis Canet - March 2017

lluis.canet@dadrin.com

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/report_hough.png "Images with raw Hought Lines"
[image2]: ./test_images_output/report_lanes.png "Images with regressed lines"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

A method called lane_detection was created, which defines the processing pipeline for an individual image. This pipeline has the following steps:

1. Convert image to grayscale
2. Apply gaussian blur to the grayscale image with a kernel of 5 pixels as a pre-processing step for edge detection
3. Perform Canny edge detection on the blurred grayscale image. The values selected for the low threshold and high threshold are 50 and 150. This values have been selected based on visual observation of the processed images.
4. Mask the edge detected image (output of previous step) by a region of interest. The region of interest has been determined visualy to be between the bottom left corner of the image, the point (450, 320), the point (490, 320) and the bottom right corner of the image. This values have been selected to mask out as many non-relevant edges as possible.
5. The final step is to peform line detection on the masked edge detected image from previous step. This step includes a configurable parameter, called "regressed". When "regressed" is False, the method will return the image annotated with the hugh lines.

In the final step, if regressed is True, then we do the following:
1. Filter lines detected where the absolute value of the slope is 0.45 (This would be lines that are abnormally close to horizontal)
2. Separate the detected lines on left and right. The criteria will for left will be all the lines with a negative slope and x1 and x2 values less than 480 pixels. The value of the slope is calculated as (y2-y1)/(x2-x1).
3. Perform linear regression and all line edges from each of the groups (left and right) and in order to calculate the slope and intercept values for each of the lines that will define the lane boundaries.
4. Calculate the edge points for the lines that will be drawn in the final images. This will be determined by the previously calculated slope and intercept values using the following formula, where y_top and y_bottom are considered the top and bottom values for y in the region of interest used for the edge detection masking
```
y1 = y_top
y2 = y_bottom
x1 = int((y1-intercept)/(slope))
x2 = int((y2-intercept)/(slope))
```
5. Once the edge of those lines are defined, they will get drawn on the image.

See the below test images processed with regressed=False.
![Images without regression][image1]

And now with regressed=True
![Images with regression][image2]

Look at the jupyter notebook on how the solution looks for the test videos.

Some additional helper functions have been added to help on the development of the solution such as ```show_image_array(...)```, which displays a matrix of images that are being inputed as an array, or ```save_image_array(...)```, which saves images inputed as an array to the path location specified in the input parameter.



###2. Identify potential shortcomings with your current pipeline

There are several shortcommings associated with this solution:
* The values selected for the edge detection parameters and the hough line detection are very sensitive to lighting conditions and will not work that well with shadows or during nightime.
* If a car crosses the lane line in front of us, it will distort the line detected.
* Since the region of interest is hardcoded, it will fail when the car is changing lanes or if the camera is placed on another location relative to the car. This will also affect if the resolution of the image changes, as it happens in the Optional Challenge.
* This lane detection will not work well on curves since the lanes will be bended.
* The current approach is very sensitve to outliers in the lane detection.
* Since each images is processed individually, there is no sequential continuity on the position of the lines which can cause the line to be very different from one frame to another.

###3. Suggest possible improvements to your pipeline

Some improvement to this solution could be:
* An adaptive region of interst based on regions on an initial processing of the image.
* Adaptive parameters for edge detection and Hough line detection based on lighting conditions.
* Segment the images for differnt lighting conditions and apply different parameters to each section.
* Detect if vehicles are covering the line lanes to avoid miss-representation of them.
* Take advantage of the sequential nature of the video by doing a weighted average of the slope and intercept parameters for the regressed lines between the current frame and the previous frames, with a decay function for the weith on the previous frames that will give smaller weight to older frames. This should help stabilize the output.

## Optional Challenge
This is an image with changing lighting conditions and shadows and therefore, the hough line parameters had to be slightly adjusted to account for the new lighting conditions. Also, the polygon that defines the area of inteset for the Canny edge detection had to be adjusted to new values, since the resolution and positioning of the camera were different than in the original scenarios.
