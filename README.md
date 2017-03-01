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

[image1]: ./output_images/calibration4_undistorted.jpg "Undistorted"
[image2]: ./output_images/test2_undistorted.jpg "Undistorted"
[image3]: ./output_images/straight_lines1_warped.jpg "Warp Example"
[image4]: ./output_images/test2_combined_binary.jpg "Binary Example"
[image5]: ./output_images/test2_hard_fitted.jpg "Fit Visual (Hard)"
[image6]: ./output_images/test2_soft_fitted.jpg "Fit Visual (Soft)"
[image7]: ./output_images/test2_result.jpg "Output"
[video1]: ./project_video.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You're reading it!

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first three code cells of the IPython notebook located in "./AdvancedLaneFinding.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the result saved in ./output_images/calibration4_undistorted.jpg

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
I used the previously defined undistort method to undistort a test image (code cell 4). The result is saved in ./output_images/test2_undistorted.jpg

![alt text][image2]

####2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image()`, which appears in code cell 5 of my python notebook.  The `warp_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```
src = np.float32([
    [580, 460],
    [700, 460],
    [1090, 720],
    [205, 720],
])

dst = np.float32([
    [260, 0],
    [1040, 0],
    [1040, 720],
    [266, 720],
])

```

I verified that my perspective transform was working as expected by applying the `warp_image()` function to a test image with straight lines and checking that after warping the image, the lines are parallel. This image is saved in ./output_images/straight_lines1_warped.jpg

![alt text][image3]

####3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (code in cell 6 of the python notebook).  The output image is saved in ./output_images/test2_combined_binary.jpg

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial

Then using the binary image and finding its histogram I chose the starting poing for my lane lines. Setting a number or sliding windows to search into and a width for them I started looking for the points that belonged to the lanes lines bottom up. Then, as a final step I fitted 2nd grade polynomials to these points. Code for this step is in cell 8, and the function that does the job is `hard_fitter()`. A result image can be found in ./output_images/test2_hard_fitted.jpg. 

![alt text][image5]

An easier way to find the lines if we've already found them is provided in the `soft_fitter()` function in cell 9. In this case we'll just look in windows around the previously found lines. An image showing the result of this method is saved in ./output_images/test2_soft_fitted.jpg

![alt text][image6]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in 11-13 in the python notebook.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 10 of the python notebook in the function `color_area()`. My result on a test image can be found in ./output_images/test2_result.jpg

![alt text][image7]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used all the transformations defined for single images (undistort, warp, color and sobel thresholds and unwarp) in every frame of the video. For each of them I performed some sanity checks comparing curvatures and offset from the current frame with the averages, for the ones that passed the checks I added their data to an array containing the last five "good" frames. I used the averages from this sanitized array to plot the green area. 

This project could be completed reasonably well using and modifying the pieces of code already provided in the lessons. The part that took more time was tuning the values of the thresholds to detect the lines. At first, some values were good enough to draw the green area, until the car gets to sections where the color of the road or the lines abruptly change to a darker or lighter one. If I had to try improving this project I'd probably start by the `combined_thresholds()` function, maybe exploring some other channels in other color spaces or fine tuning the sobel thresholds and parameters.