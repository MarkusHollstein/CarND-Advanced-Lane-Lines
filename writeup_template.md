## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration5.jpg "Undistorted"
[image2]: ./output_images/test2.jpg "Road Transformed"
[image3]: ./output_images/test3_binary.jpg "Binary Example"
[image4]: ./output_images/straight_lines_transf.jpg "Warp Example"
[image5]: ./output_images/test2.jpg "Fit Visual"

[image6]: ./output_images/test2.jpg "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!





### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second cell of the IPython notebook located in "untitled.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
As above I just use cv2.undistort with the matrix and dist calculated above.


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In the first cell after the headline "Thresholded Image" I define the functions I combine to get a thresholded image.
In the next cell I use a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Under the headline Perspective transform I first used an image of straight lines to determine  source points (called pts) that shall be mapped onto a rectangle by my transformation. Therfore I used points of the lane lines at the bottom  of the image and some way up. I visualized the polygon on the image.
In the next cell within the function persp_transf I define destination points using an offset and calculate the transforamtion matrix.
In the function transform I combine unditortion with the perspective transform. This leads to the following picture:


![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:
I identified line pixels by first using a histogram to find the starting point for the left and right line. After fixing the parameters I regarded the nonzero pixels within rectangles with bottom center at the above defined positions and depending on the postions of the nonzero pixels I moved the rectangles above the to the right or left (compared to the one at the bottom). The nonzero points within these rectangles are saved and returned by the function.
These are then used in the function fit_polynomial to calculate the coefficients of a second order polynomial and calculate other things like the curvature and the position of the car.
This is a visualisation of my fitted polynomial

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 15 through 30 the function fit_polynomial. I used the formula from the lecture to calculate the curvature and transformed it back to meter (instead of pixels) by using the fact that a lane is about 3.7m wide in my transformation it was 700 pixels wide so I used the factor 3.7/700 to transform the x coordinate. (Similarly for y)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the cell above the headline video.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I used the class Line proposed in tips and tricks. I used a right and a left line as global variables and using a new image I updated the Lines if the values for the new lines that were detected in the image were reasonable (I decided to calculate the width of the lane at the bottom of the image and at the upper end of the detected area and see if it is within a range of the 3.7m that are the minimal width of a lane). For non-reasonable values I decided to keep the old values from a well detected image. Besides having discovered good lines I used them to only take a certain area around them in the next image and ignore all the rest (done in the function helper in the third cell after the headline pipline).

There are certainly many things to be improved. For example the processing does not work if the lines are not well detected in the first image of the video. Besides shadows and other unusual marks on the road (as in the challenge videos) are still a problem. To use it in a real car the computation takes too long so the current lines would be calculated too late.
