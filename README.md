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
[image1]: ./output_images/calibration.png "Calibration"
[image2]: ./output_images/road_transform.png "Road Transformed"
[image3]: ./output_images/rg.png "Threshold 1"
[image4]: ./output_images/combo.png "Threshold 2"
[image5]: ./output_images/warped.png "Warp Example"
[image6]: ./output_images/color_fit_lines.png "Fit Visual"
[image7]: ./output_images/output.png "Output"
[image8]: ./output_images/histogram.png "Histogram"
[video1]: ./output_videos/project_video_output.mp4 "Video"

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in `./pipeline.ipynb`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()`.


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (please refer the thresholding steps in `get_binary()` function in the code block 2 of `pipeline.ipynb`).  Here's an example of my output for this step. 

![alt text][image3]
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code block two in notebook`pipeline.ipynb` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I first applied histogram to the warped binary image. After applying histogram, I found the max peak of the pixels at two points. I considered them as Left and Right base points.  Then I used a sliding window to search the lane pixels step by step by going through each rows.

![alt text][image8]

After I found the lane line pixels (non-zero pixels), separated them according to their region as left or right and I stacked them up with the previously found pixels and completed for the entire image frame. After that I separated the x and y position for each pixel in the right and left lane line.

Then I fitted the x and y to form a 2nd order line. To fit line, I used equation similar to this:
`left_fitx = left_fit[0]*ploty**2 + left_fit[1]*ploty + left_fit[2]` Since the line is not a straight one, we cannot use 1st order polynomial for fitting the curve. Hence I used a 2nd order polynomial.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the function `fit_lines()` in my code in `pipeline.ipynb`. The code itself is pretty self explanatory with the comments included inline.
I used the following formula to convert the line fittings from pixel space to real world space (in meters)

> ym_per_pix = 30/720 
> xm_per_pix = 3.7/700

After fitting the line to real world space, I found the radius of curvature for left line fit in the line :
`left_curverad =  ((1 + (2*left_fit_cr[0] *y_eval*ym_per_pix + left_fit_cr[1])**2) **1.5) / np.absolute(2*left_fit_cr[0])`

Then I similarly calculated for the right line fit. After getting radius of two lane lines, I averaged them so that we get mean radius of curvature.

Following that I found the offset of the car from the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `addWeighted()` and `warp()` function in `pipeline.py`.  The `warp()` function can do both unwarping and warping by passing additional parameter `warp=False`.

Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I used machine learning approach to detect the lane lines. I started off with calibrating the camera. Once the camera was calibrated, I used the calibration matrix to undistort the image frames from the video. 

Then I used a combination of gradient and color thresholding to detect the lines. I used sobel function to threshold because it is very flexible when compared to Canny. We can specify the direction of the gradient. Along with this I used color threshold. For this, S channel was my preference since it is very less likely to be affected by the sunlight and varying contrasts. After this I combined both the threshold to get the final threshold binary.

To detect the curvature of the lane lines, I transformed the image into bird's eye perspective. After that I used histogram and sliding windows to detect and fit the lane pixels on the perspective. After fitting the line, I unwarped the line fit to the real world perspective and plotted over the original image.

I used 2nd degree polynomial for the curve line fit and used the formula provided by Udacity to convert from pixel space to real world space.

For video, I created pipeline and used global variables for the buffer instead of using class implementation.

> Difficulties faced

First thing was thresholding parameters. It is very crucial for the lane lines to be detected correctly. I spent most of the time tuning the thresholding parameters. The other thing was, 'what channels to combine?'. Finally I went with S channel threshold and x gradient threshold.

At first my pipeline was working very fine for all images I fed. But while in video, it sometimes failed to detect the line markings. Then I fine tuned my video pipeline to work with video also.

> Needs improvement

I know, this is not a robust model. Few jitters and wobbling is seen in the video. I will try to make the lane detection even smooth in all the challenge videos provided. If possible, I'll try with a video recording of mine over this weekend.

> Best regards, 
> Vivek Mano
