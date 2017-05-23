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

[image1]: ./output_images/marker_detection.png "Chessboard Corners detection"
[image2]: ./output_images/chessboard_undistorted.png "Chessboard Undistortion"
[image3]: ./output_images/test_image.png "Sample Test Image"
[image4]: ./output_images/undistorted_image.png "Undistorted Image"
[image5]: ./output_images/unwarped_image.png "Unwarped Image"
[image6]: ./output_images/sobel_abs.png "Sobel Absolute"
[image7]: ./output_images/sobel_mag.png "Sobel Magnitude"
[image8]: ./output_images/sobel_dir.png "Sobel Direction"
[image9]: ./output_images/sobel_mag_dir.png "Combined Sobel Magnitude + Direction"
[image10]: ./output_images/color_channels.png "Colorspace Channels"
[image11]: ./output_images/HLS_L_Channel.png "HLS L-Channel"
[image12]: ./output_images/LAB_B_Channel.png "LAB B-Channel"
[image13]: ./output_images/piepline_all.png "Run Pipeline on All Image"
[image14]: ./output_images/sliding_window_polyfit.png "Sliding Window Polyfit"
[image15]: ./output_images/sliding_window_histogram.png "Sliding Window Histogram"
[image16]: ./output_images/previous_fit_polyfit.png "Previous fit Polyfit"
[image17]: ./output_images/lane_draw.png "Lane drawn on original Image"
[image18]: ./output_images/curve_draw.png "Curvature Data drawn on original Image"

[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/abhardwajnv/CarND-Advanced-Lane-Lines/blob/master/writeup_report.md) is a writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

First code cell includes all the imports required to run the project successfully.
The code for this step is contained in the second & third code cells of the IPython notebook located in root folder `./Advanced_Lane_Lines.ipynb`.  

To start with calibration process we need images covering (x, y) coordinates, skew and sizes. There were 20 input images taken as reference for corner detection.
I start by preparing NULL `object points` array, which will be the (x, y, z) coordinates of the chessboard corners in the real world space [3D] & NULL `image points` array, which will be the (x, y) pixel positions of each corner in the image plane [2D]. Here the chessboard for all the images is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image which is achieved by by using `cv2.findChessboardCorners()` function. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Using `cv2.drawChessboardCorners` function of OpenCV i drew the corners on the original images and obtained following resulting images:

![Chessboard Corners Detection][image1]

You can see 3 images are shown blank because the chessboard corners were not detected in those images.


I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function. I took one image for reference from the 20 input images and applied the calibration & undistort functions on it and obtained following result:

![Chessboard Undistortion][image2]

You can clearly make out the difference in the original image and undistorted image where distortion effect is not visible and the image is flat.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in 5th code cell of IPython notebook.
Starting with the first step of the complete pipeline i created one function named `undistort()` which in turn used OpenCV `cv2.undistort` function to undistort the input image. To Visualize the effect of undistort function i took one image for reference from given set of test images `test3.jpg` and applied this function to the image.
The output of the function is shown below:

![Undistorted Image][image4]

In this image its a bit difficult to make out the difference at first sight of undistortion but observing the left, right and bottom areas of image can give idea of how undistortion has worked on the original image. Check the shape of the trees on the left and shape car hood at the bottom which gives clear identification of undistortion.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I started with implementing the gradient thresholds explained in Udacity lectures (Sobel Absolute/Sobel Magnitude & Sobel Direction) also trying with combination of these thresholds. Code for this part is contained in 7th to 10th Code cells of IPython notebook.

Below are the images showing different gradients applied on the input image.

Sobel Absolute.

![Sobel Absolute Gradient][image6]

Sobel Magnitude.

![Sobel Magnitude Gradient][image7]

Sobel Direction.

![Sobel Direction Gradient][image8]

Sobel Magnitude+Direction.

![Combined Sobel Magnitude + Direction Gradient][image9]

In addition i started with using the HLS & HSV color spaces to run in my pipeline. Although L Channel was working appropriately for the white lanes detection, i read more "http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html" about the LUV & LAB color channels where B Channel showed more accurate results for the yellow lane lines. Code for this part is contained in 11th to 13th code cells of IPython notebook.

Below is image which shows various color channels with different color spaces for input image.

![ColorSpace Channels][image10]

With default gradients and HLS/HSV color spaces i was not able to get the lane detection work properly in different lighting conditions like shadow's, bright light on the edges and that was getting rectified with the use of B-Channel hence i decided to only keep the L-Channel threshold for isolating white lane lines and B-Channel threshold for isolating yellow lane lines.

Below are the images showing binary image output of both the color space thresholds.

HLS L-Channel: I used threashold of (220,225) for L Channel.

![HLS L-Channel][image11]

LAB B-Channel: I used threashold of (190,225) for B Channel.

![LAB B-Channel][image12]

Then i defined the `run_pipeline()` function containing binary threshold pipeline for image processing which included following steps.
1. Undistort Image.
2. Unwarp Image.
3. Apply HLS L-Channel threshold
4. Apply LAB B-Channel threshold
5. Combine the color channel thresholds for steps 3 & 4.
6. Return the binary output image.

Code for this pipeline is contained in 14th code cell of IPython notebook.

Then i ran this pipeline on all the test images available and following is the output of same which is contained in 15th code cell of IPython notebook.

![Run pipeline on all images][image13]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in 6th code cell of IPython notebook. The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

| Source        | Destination   |
|:-------------:|:-------------:|
| 580, 450      | 450, 0        |
| 750, 450      | w-450, 0      |
| 250, 680      | 450, h        |
| 1150, 680     | w-450, h      |

where h,w (referring to height and width) were taken from the image shape as `h,w = img.shape[:2]`.

Using OpenCV function `cv2.getPerspectiveTransform` i deduced the transform matrix `M` and inverse matrix 'Minv' and then warped the image to Birds View using `cv2.warpPerspective` function of OpenCV using the source & destination points and the transform matrix generated in earlier step.
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Below image shows the output of the warped image:

![Unwarped Image][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Code for this section is contained in 16th to 18th code cells of IPython notebook.

So far we generated binary images which were first undistorted then transformed using unwarp function and then applied with color channel thresholds.
As we saw in Udacity chapeters that sliding window polyfit helps in identifying the lane-line pixels which were not identified so far with the ealier pipeline.

So i created 2 functions named `sliding_window_polyfit` and `prev_polyfit`. These functions identified the lane lines and fit a second order polynomial for both left & right lanes. Sliding window polyfit identifies the base x points for left and right lanes from the local maxima of the left and right quarters of the histogram. It identifies 10 windows from which to identify lane pixels, each one is centered on the midpoint of the pixels from the window below it. This identifies the lane lines effectively from bottom to top on the binary image as input.It also helps accelerating the process by just searching for activated pixels over a small portion of image. Pixels associated with each line are identified using numpy function `polyfit()` which fits a second order polynomial to each set of pixels.

Below image shows the output of sliding window polyfit on binary image.

![Sliding Window Polyfit][image14]

Below image shows the histogram generated by sliding window polyfit function which returned base points for left and right lanes.
There are peaks visible in the histogram which are closest to center.

![Sliding Window Histogram][image15]

The other function `prev_polyfit` helps in the search process by leveraging the fit from last frame and only applying the search for lane pixels in a certain range of that fit. Below image shows the output of this function which shows the shaded area from previous fit and internal lines are applied on the current frame.

![Previous Fit Polyfit][image16]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Code for this section is contained in the 19th code cell of IPython notebook.

I used the reference radius of curvature formula to compute the radius which is explained as below.
The formula for the radius of curvature at any point x for the curve y = f(x) is given by:

Radius of Curvature = [1 + (dy/dx)^2 ]^3/2 / |d^2 x / dx^2|

This was calculated in the program as shown below.

```
lcurve = ((1 + (2*lfit_cr[0]*y_eval*y_metppix + lfit_cr[1])**2)**1.5) / np.absolute(2*lfit_cr[0])
rcurve = ((1 + (2*rfit_cr[0]*y_eval*y_metppix + rfit_cr[1])**2)**1.5) / np.absolute(2*rfit_cr[0])
```

Here lfit_cr[0] & rfit_cr[0] are the first coefficients of the second order polynomial and subsequntly lfit_cr[1] & rfit_cr[1] are the second coefficients.
y_eval is the bottom most y position in the image. And y_metppix is the factor used for the conversion of pixels to meters.

In Addition the position of vehicle is calculated using below formula:

```
lane_center_position = (r_fit_x_int + l_fit_x_int) /2
cdist = (car_position - lane_center_position) * x_metppix
```

Here r_fit_x_int & l_fit_x_int are the x-intercepts of the right and left fits respectively. Car position is the difference between the image midpoint and these intercepts.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 20th and 21st code cells of IPython notebook in `draw_lane()` & `draw_rad_dist` functions which as the name suggests, first draw lane lines on the original image and second draws the radius of curvature and distance from center on image.

Here is how the test image looks after these operations:

![Lane drawn on image][image17]

![Curvature & Distance drawn on image][image18]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most difficult part initially was to tackle with the different lighting conditions on the track like shadows, bright lights due to which applying different thresholds were getting a tedous task. Although after i came across the LAB color space and saw the effect of B-Channel on the frames i was able to improve the pipeline and get the lane detection pipeline working for the project_video.mp4. Before this when i applied the absolute gradient threshold and L channel the lanes were getting shaky on the curves and under different light changing points like shodow under the tree and the bright sunlight area. This got resolved with my final pipeline.

Although i'm still not very clear with the fundamentals of different color spaces so i would learn about more different color spaces and how they behave for different types of lighting conditions as well as different lane colors.

With this the pipeline effectively worked for the project video but still there are some wobbles which can be seen throughout the video and mostly at shadowed area.
I tried this same pipeline on other two chanllenge video's as well but it failed miserably. This is another taks i would like to imrove in future.

* Points where my pipeline is likely to fail:
Mostly in the case of very sharp lighting conditions and different weather conditions this pipeline will fail.
In case there is more noise in the pixels adjacent to the detection this pipeline will fail.

* Improvements:
To improve more on this pipeline i will work on different gradient thresholds and different color spaces to cover most possible conditions.
To start with i will make the project working on the challenge video's.  
