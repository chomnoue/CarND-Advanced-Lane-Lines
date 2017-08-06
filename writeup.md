## Writeup Template

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

[undistort_output]: ./examples/undistort_output.png "Undistorted Output"
[original]: ./examples/original.png "Original Image"
[undistorted]: ./examples/undistorted.png "Undistorted"
[color_gradient_combined]: ./examples/color_gradient_combined.png "Color Gradient Combined"
[warped]: ./examples/warped.png "Warped"
[slide_windows]: ./examples/slide_windows.png "Slide Windows"
[reuse_fit]: ./examples/reuse_fit.png "Reuse existing fit"
[final]: ./examples/final.png "Final Image"

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

All the code referenced in this report can be found in the [advanced_lane_line_finding.ipynb](./advanced_lane_line_finding.ipynb) notebook.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients (see `cal_undistort()` in the notebook) using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistort_output]

### Pipeline (single images)

Here I will show how the following image has been transformed through each step of the pipeline:
![alt text][original]

#### 1. Provide an example of a distortion-corrected image.

Applying the cal_undistort() function describe above, I get the following undistorted image:
![alt text][undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (see `combine_color_and_gradient()` function in the notebook).  Here's an example of my output for this step. (The four colored points on the image are actually the source points used in the perspective transformation described below)

![alt text][color_gradient_combined]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`,  (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[200, height], [535,480], [745, 480], [1100, height]])
dst = np.float32([[320, height], [320, 0],[980, 0],[980, height]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 320, 720      | 
| 535, 480      | 320, 0        |
| 745, 480      | 980, 0        |
| 1100, 720     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the above image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

##### Slide windows and fit plolynomials for new images

For a new warped image, I use the `slide_windows_and_fit_polynomials()` function to fit two second order polynomials for the left and right lane lines. It does the following:
	1. Take a histogram of the bottom half of the image
	2. Find the peak of the left and right halves of the histogram, to be used as starting positions
	3. Divide the image vertically in n slide windows 
	4. For each window:
		* Draw a small rectangle around each starting position (left and right)
		* Find the indices of non-zero pixels in the rectangle
		* If the number of good indices are greater than a minimum, change the starting position to their center
	5. Fit the second order ploynomila for the set of found indices

The following image is the result of the above algorithm applied on the warped image. The searched zones are drown in green, and the fitted plolynomials in yellow:

![alt text][slide_windows]

##### Reuse existing fits if any

If I have existing second order ploynomials from previous image, I apply the `fit_polynomials_with_previous_fit()` function. Instead of a blind search, it just searches in a margin around the previous line position.
Following is the results of applying such search on the warped image. The Search zone appears in green in the image:

![alt text][reuse_fit]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Notice that from each function described in the above section, I fit, for each line, both an image space second order plolynomial (in pixels) and a world space second order polynomial (in meters). You can see the code in the `compute_fits()` function.
I used the world space second order polynomial to compute the radius of curvature (see `curvature()` function)

The position from the center is calculated with the assumption that the camera is mounted at the center of the car (see `dist_from_center()` function).


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I put all the previous steps in the function `pipeline()` to transfrom each image form end to end.  Here is an example of my result on the test image:

![alt text][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The described pipeline worked fine on most of the project video, with some wobbly lines in few sections of the video. I should try to fine tune the sanity checks and usage of the results on previous images to make it smoother.

The result on the chanllenge video is catastrophic. You can see it in [challenge_video_output.mp4](./challenge_video_output.mp4)
 
 1. Sometimes the shadow line in the midle of the road is detected as the ritht lane line. I think that this can be fixed by 
 2. The found lines are not even parallel under the bridge

So in the nex iteration, I plan to do try following
 
 1. Add images from challenge video to my test images
 2. Improve the color and gradient combination to be able to distinguish the lines under shadow
 3. Restrict the range of the histogram where the peaks are choosen in the `slide_windows_and_fit_polynomials()` function to try ignoring noisy lines.