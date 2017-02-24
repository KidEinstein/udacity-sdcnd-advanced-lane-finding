# **Advanced Lane Finding Project**

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

[image1]: ./output/undistorted_image.jpeg "Undistorted"
[image2]: ./output/undistorted_road_image.jpeg "Road Transformed"
[image3]: ./output/binary_combo.jpeg "Binary Example"
[image4]: ./output/warped_straight_lines.jpeg "Warp Example"
[image5]: ./output/fit_lines.jpeg "Fit Visual"
[image6]: ./output/example_output.jpeg "Output"
[video1]: ./output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "AdvancedLaneFinding.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
The camera matrix and distorting coefficients obtained using the chessboard images are used the undistort the road images. The output for one of the images is shown below.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. I generated heatmaps for the H, L and S components of the road image to determine which component is most helpful in demarcating the lanes. The S component seemed the most promising. Using color thresholding for the S component gave good results for most test images.

Gradient thresholding was applied on S component to pick up the edges of the lane lines.

Using a combination of color and gradient thresholding gave good results. A sample image is shown below.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective()` which uses OpenCV's `getPerspectiveTransform()` function to get the tranformation matrix and `warpPerspective()` to perform the tranformation on an image. The following source and destination points were chosen:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 240, 686      | 300, 700        | 
| 1060, 686      | 1000, 700      |
| 738, 480      | 1000, 300        |
| 545, 480     | 300, 300      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I fit a second degree polynomial on the warped image obtained in the previous step, the code to fit the polynomial is defined in the `fit_poly()` function. For the first frame, the polynomial is obtained by starting at the bottom of the image and identifying where the lane lines lie by observing the peak of the histogram for the identified lane line in the left and right half of the road image. The algorithms then progressed upwards in the image in determines the other points which lie on the lane. At the end, Numpy's `poly_fit()` function is used to obtain the coeffiecient of the polynomial, which is then plotted on the road image.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to find the radius of curvature can be found in the `fit_poly()` function. The lane was assumed to be about 30 meters long and 3.7 meters wide. First the polynomial obtained for the warped image was used to generate a polynomial for the world space, the the formula for radius of curvature for a polynomial was used to calcluate radius of curvature at the bottom of the image.

For finding the distance from lane center, the camera was assumed to be placed at the center of the car and the lane center was obtained by finding the mid point of the the left and right lane lines at the bottom of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the `fit_poly()` function.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My initial implementation had an issue with frames in which the lane lines were wrongly identified. This happened only for a few frames and to counter the issue, I maintained the last thirty polynomials and used the average of their coefficients to obtain the best fit polynomial. This approach alleviates the problem to a certain degree. To make the pipeline more robust, I would like to include sanity check to determine if the polynomial fit for a fit is wrong and discard it and fall back to the previous best fit. My pipeline would likely fail if there are series of misidentified frames.
