##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/Undistort.png "Undistorted"
[image2]: ./test_images/test3.jpg "Road Transformed"
[image3]: ./output_images/undistort_on_TestImage.png "Undistort on Test Image"
[image4]: ./output_images/Binary_onwarp.png "Binary Warp"
[image5]: ./output_images/Histogram.png "Histogram"
[image6]: ./output_images/Sliding_Window.png "Sliding Window"
[image7]: ./output_images/Polyfit_sliding_window.png "Sliding Window"
[image8]: ./output_images/Final_detected_lane.png "Sliding Window"
[radius]: ./output_images/radius.png "Radius Formula"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "P4_Submit.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I used following code:
'''
image = mpimg.imread('./test_images/test3.jpg')
undist = cv2.undistort(image, mtx, dist, None, mtx)
'''

The code for this step is contained in the fourth cell of Ipython notebook. We first read the image using mpimg.imread and then use cv2.undistort() to undistort the image. The undistorted image can be seen as below: 
![alt text][image3]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. I gray scaled the image using cv2.cvtColor() and then calculated its Laplacian. Then I applied gradient thresholds to the image. For color thresholding, I used the s-channel, and created a binary image. Finally, I combined the gradient and color thresholds. Here's an example of my output for this step.

![alt text][image4]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cells 5 through 6 in P4-SDCND.ipynb). I gray scaled the image using cv2.cvtColor() and then calculated its Laplacian. Then I applied gradient thresholds to the image. For color thresholding, I used the s-channel, and created a binary image. Finally, I combined the gradient and color thresholds. Here's an example of my output for this step.

   The code for my perspective transform includes functions called find_perspective_points() and get_perspective_transform. The find_perspective_points() function takes as inputs an image (img), and generates appropriate source (src) and destination (dst) points. The function get_perspective_transform() takes as input an image (img) and source (src) and destination (dst) points, and generates the M and Minv matrix. These matrices are then used to get the warped image using the function cv2.warpPerspective().

If the function find_perspective_points() fails to find source (src) and destination (dst) points, then I chose to hardcode the source and destination points in the following manner:
'''
src = np.array([[585. / 1280. * img_size[1], 455. / 720. * img_size[0]],
                        [705. / 1280. * img_size[1], 455. / 720. * img_size[0]],
                        [1130. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [190. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)

dst = np.array([[300. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [1000. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [1000. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [300. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
'''

I read this approach on one of the forum discussion and tried to implement it.

I verified that my perspective transform was working as expected by plotting the original image and warped image and verifying that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After getting the binary image we try to find the histogram. The peaks in the histogram indicate the lane lines:

![alt text][image5]


With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. The sliding window approach is implemented in the function sliding_window(). After implementing sliding window I used the function fit_searched_windows() which calculates the area in which we should search lane lines. My ouptput looks as follows:

![alt text][image6]


![alt text][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Once we have second order polynomial fit we can calculate the radius of curvature. The formula used to caluculate is as shown below:

![alt text][radius]

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 13 in my code file `P4_Submit.ipynb` in the function `proccess_image()`.  Here is an example of my result on a test image:

![alt text][image8]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion 

####1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

 Some of the issues that I faced are as follows: 
* I tried applying different color thresholds and they gave different results. Choosing a correct value of thresholf to convert to binary seems to be very important.
* I tried different colors spaces as well. RGB to YCbCr, HSV and HLS. Each color space gives different information.I might plan to use different color spaces dynamically to obtain lanes in future.
* Choosing points for initial perspective transform was a big headache.
*  My algorithm jitters a little at initialization.

To make it more robust, I could do following things: 
* Smoothing techniques can be used to obtain better results. 



