## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
---

The goal of this project is to write a software pipeline to identify the lane boundaries in a video. In addition to the code this writeup will provide a detailed explanation of the process used for lane detection. 

**Project Goals**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

**Project Files**

The repository consists of the following files:
- Main.ipynb (Notebook containing process breakdown and the pipeline)
- mtx_dist_pickle.p (Pickle file containing calibration parameters)
- project_video.mp4 (Video used for lane detection)
- project_video_output.mp4 (Output video generated with lane detection)
- README.md (Detailed description of lane detection process)

[//]: # (Image References)

[image0]: ./examples/Undistort_example.png "chessboard"
[image1]: ./examples/Undistort_example_2.png "Undistorted"
[image2]: ./examples/threshold_example.png "Binary Image"
[image3]: ./output_images/02_test_warped_img_0.jpg "Warp Example"
[image4]: ./examples/histogram.png "Histogram"
[image5]: ./output_images/04_test_window_img_0.png "Window Example"
[image6]: ./output_images/05_test_result_img_0.jpg "Output"
[video1]: ./project_video_output.mp4 "Output Video"
[image7]: ./camera_cal/calibration1.jpg "Calibration"
[image8]: ./examples/Rcurve.png "Rcurve Formula"

## Dataset

Folders:
- `camera_cal` : Images for camera calibration
- `test_images` : Images for testing your pipeline on single frames
- `project_video.mp4` : Video the pipline is tested on
- `output_images` : Results of test images

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "./main.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function.

Calibration Image:
![alt text][image0]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

Original Image:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell 3 function `combined_threshold()`). My process involves applying the sobel operator along the x-axis followed by V and S color channel selection from HSV and HLS color space. Finally I applied magnitued and directionaly thresholding which resulted in the output binary image. 

Here's an example of my output for this step.  (note: the image below is a result of performing threshold on a warped image of the original Test Image 3 shown above)

![alt text][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective transform()`, which appears in Cell 3 of my Ipython notebook. The `warper()` function takes as inputs, as well as source (`src`) and destination (`dst`) points and returns matrix M and Minv. Using the M matrix I perform the openCV `cv2.warpPerspective()` function to receive the warped image. I chose to hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 525, 464      | 460, 0        | 
| 630, 464      | 804, 0        |
| 300, 682      | 460, 720      |
| 700, 682      | 804, 720      |

I verified that my perspective transform was working as expected by looking at the `src` and `dst` points on a test image and its warped counterpart to verify that the lines appear about parallel in the warped image.

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once I had the warped image, I took a histogram of pixels, and the peaks helped me locate the starting points of lanes. The histogram of one test image can be seen below:

![alt text][image4]

Then I used np.polyfit function to fit my lane lines with a 2nd order polynomial. Next I used the sliding window approach the code for which can be found in cell 13 of my ipython notebook. This resulted in the following output image:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented this step in cell 15 using the function `find_lane_curvature()` which is defined in cell 3 of my ipython notebook. There is another function called `dist_from_center()` which helps find the deviation from center. The formula I used for this is as follows:

![alt text][image8]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

My final pipeline function `pipeline()` can be found in cell 18 of my ipython notebook. Here is an example of my result on a test image:

![alt text][image8]




---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most challenging part of this project involved tuning the threshold parameters and getting the warping coordinates right such that the lines appear parallel in the warped image.

The pipeline would fail if more than 1 lane appears in an image, for example when changing lanes because it was presumed only two lanes exist for the purpose of this project. 
