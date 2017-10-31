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

[image1]: ./examples/cameraCaliberation.PNG "Caliberated Chessboard"
[image2]: ./examples/undistort.PNG "Undistorted"
[image3]: ./examples/threshold1.PNG "Threshold1"
[image4]: ./examples/threshold2.PNG "Threshold2"
[image5]: ./examples/warped.PNG "Warped image"
[image6]: ./examples/warped_straight_lines.jpg "Warp Example"
[image7]: ./examples/color_fit_lines.jpg "Fit Visual"
[image8]: ./examples/roc.PNG "ROC"
[image9]: ./test_images/test3.jpg "Test 3 Image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "https://github.com/aarti9/CarND-Advanced-Lane-Lines/blob/master/AdvanceLaneDetect.ipynb" .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  I used cv2.findChessboardCorners() for finding chessboard corners (for an 9x6 board) for grayscale image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Example of the chessboard corners detected image is as below:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the second code cell of my IPython notebook. To demonstrate this step, I will describe how I apply the distortion correction to one of the test images (test3.jpg and one of the chessboard image):

 I applied this distortion correction to the test image and a chessboard image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

You can notice the curvature of the chessboard on the left side is corrected. The test image of the road is already fine, hence we do not see any visible correction. But the idea is to correct in case it is required.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the third code cell of the IPython notebook. I experimented with different thresholding mechanisms like mag_threshold which gives magnitude of the gradient and abs_sobel_thresh. Also tried various mask filters for white and yellow line detection. Finally I combined white and yellow masks to get the lane as left side lane had yellow color.

Here's an example of my output for this step. 

![alt text][image3]

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the fourth code cell of the IPython notebook. I chose the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203.3, 720      | 320, 720      |
| 1126.6, 720     | 960, 720      |
| 695, 460      | 960, 0        |


![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I followed the instructions from our lecture material. For the first frame, I used the histogram to find out the peaks for the left and right side of lane lines. Tried to recentre sliding window based on left and right pixel position and a margin. Then I fit a second order polynomial using numpy's polyfit function. For subsequent frames, we continue using the left_fit and right_fit value we used when fitting the second order polynomial in first frame. Then I generated a polygon to illustrate the search window area and recast the x and y points into usable format for cv2.fillPoly(). We use cv2.addWeighted methods to add two images and assign appropriate weightage.
We process each frame in the video clip input video and then produce the output video with land detected and marked in green

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines #147 through #154 in my code. I used the technique used in lecture material to calculate left_curverad and right_curverad. I printed left_curverad and distance from center position values on the test image, for which I got some ideas to show overlap text on the image from internet. The example is shown in section 6 below which also illustrates the plotting back of lane identified

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines #157 through #169 in my code to show the lane lines identified and filled with green. Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had few issues with the thresholding. I tried to combine various thresholds, but for few combinations it was resulting in blank image. Finally only the white and yellow masking threshold worked, which was fine as that helped in identify the lane. 
A potential issue might be when the lines are not marked properly or even in first frame we fail to identify the left and right lines, or some object comes on the path in front of the car which hides the lane. I think we might have to also take help from GPS navigation system tracker of where we are and how does front road looks like and use that to augment our program.
