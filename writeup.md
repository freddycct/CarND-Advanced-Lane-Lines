## Writeup

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

[video1]: ./project_video.lanes.mp4 "Video"

[distortion_correction]: output_images/distortion_correction.png "before and after correction"
[original_undistorted]: output_images/original_undistorted.png "before and after correction"
[perspective1]: output_images/perspective1.png
[perspective2]: output_images/perspective2.png
[binarize_warp]: output_images/binarize_warp.png
[warp_binarize]: output_images/warp_binarize.png
[grayscale_or_schannel]: output_images/grayscale_or_schannel.png
[grayscale_or_schannel1]: output_images/grayscale_or_schannel_1.png
[binarize_images]: output_images/binarize_images.png
[a]: output_images/a.png
[b]: output_images/b.png
[polynomial]: output_images/polynomial.png
[final]: output_images/final.png

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./advance_lane_finding.ipynb", under the function 'calibrate_camera'.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![distortion_correction]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![original_undistorted]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I do the perspective warping first because I am curious about whether we should 
1. warp first then apply color+gradient thresholds .
2. or apply color+gradient thresholds then warp the image.

I hardcode the source points in the following manner using this image (look for the red dots) ![perspective1]:

```python
src = np.float32([
  [260,680], # bottom left
  [1040,680], # bottom right
  [595,450],
  [685,450]
])

dest = np.float32([
  [260,720],
  [1040,720],
  [260,0],
  [1040,0]
])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 260, 680      | 260,720       | 
| 1040, 680     | 1040, 720     |
| 595,450       | 260,0         |
| 685,450       | 1040,0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![perspective2]


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

This section determines wheher we should warp perspective then binarize the image or binarize the image then warp the perspective of the image.

I used a combination of color and gradient thresholds to generate a binary image.

Refer to function filter_colors_hsv for the color thresholds to detect yellow and white lane lines.

First I try to warp then filter colors and binarize it, I obtain the following:
![warp_binarize]

Then I filter colors, binarize it then warp it, I obtain the following:
![binarize_warp]

It is not clear which sequence is better, so now I work on the gradient thresholds.

For gradient thresholds, there is a decision of whether we should apply the sobel filter on the grayscale image or the s channel of the image from HSV space.
![grayscale_or_schannel]
![grayscale_or_schannel1]
After applying sobel filter in (x & y direction) on the grayscale and schannel, I zoomed in on the lane lines to determine which is a better choice. So it seems that the sobel filter works very clearly in the S channel.

Then I zoom further in on the two images above and determined that a lower threshold of 20 on the pixel values is sufficient to detect the lane lines. These are the outcome of the binary images.
![binarize_images]
I determine that the sobel filter on the x direction is sufficiently good enough.

Finally, back to the issue of whether to warp then binarize or binarize then warp, I looked at the sobel gradients for this two images:
![a]
![b]

So I determine it is better to binarize then warp.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

![polynomial]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result]![video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
