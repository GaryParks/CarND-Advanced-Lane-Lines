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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/test.png "Road Transformed"
[image3]: ./output_images/test_out.png "Undistorted"
[image4]: ./output_images/hls_s.png "HLS"
[image5]: ./output_images/color.png "Color"
[image6]: ./output_images/perspective.png "Birds Eye"
[image7]: ./output_images/poly.png "Window Polyfit"
[image8]: ./output_images/prev.png "Previous Fit"
[image9]: ./output_images/form.png "Formula"
[image10]: ./output_images/final.png "Final Fit"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first, second, and third code cells of the IPython notebook located in ".P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used color and thresholds to generate a binary image (in blocks 8 - 10).  I first tried looking at the S channel of HLS as shown in the lecture:

![alt text][image4]

But in a few images, the lanes were not as clear as I thought that they should be, and especially in images 7, the shadows caused some issues.  So I decided to look at more color spaces.  The results can be seen in blocks 8 and 9, and from this I decided that the LAB color space was best, esoecially since one channel is related to yellow.  I chose to use LAB's B-channel for the yellow lane markers and the L-channel for white lane markers.  Then I combined the result into one image like shown:

![alt text][image5]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birdseye()`, which appears in the 4th code cell of the IPython notebook.  The `birdseye()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 450, 0        | 
| 250, 680      | 450, h      |
| 1060, 680     | w-450, h      |
| 687, 460      | w-450, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Code for this section is contained in the 4th to 11th code cells of IPython notebook.

I made 2 functions named window_polyfit and previous_polyfit. These functions fit a second order polynomial for both left & right lanes. 

The window polyfit function identifies the base x points for left and right lanes from the local maxima of the left and right quarters from the center of the histogram. It identifies 10 windows where each one is centered on the midpoint of the pixels from the window below it. It helps by decreasing the search area to a small portion of image. Here is an image of the functions results with the corresponding histogram.

![alt text][image7]

The other function previous_polyfit uses the fit from the last frame as the search area for the next frame.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Code for this section is contained in the 4th code cell of IPython notebook defined in the curvature function.

![alt text][image9]

~~~
lcurve = ((1 + (2*left_fit_cr[0]*y_eval*y_metppix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
~~~

In Addition the position of vehicle is calculated by looking at the left and right x-intercepts and comapring it to the center of the image.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code block 4 in my code in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I decided to look at different color spaces for my first try since it was easier for me to understand the different colorspace outputs and the images that they were tested on.  I ended up not using any Sobel thresholds since it seemed like the performance from the LAB color space good.  I would be interested to see how it would perform in more complicated situations like on and off ramps.  I would look in to the Sobel thresholds more as a start if I needed to improve the performance.

For some reason I was not able to test my code on either of the challenge videos.  When creating the output videos, it finished the project video but halfway throught the first chanllenge video something crashed.