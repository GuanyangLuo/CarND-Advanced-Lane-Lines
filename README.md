## Project 2 Writeup

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
[image2]: ./output_images/test1/distortion_corrected_image.jpg "Undistort Example"
[image3]: ./output_images/test1/binary_combo.jpg "Binary Example"
[image4]: ./output_images/test1/warped.jpg "Warp Example"
[image5]: ./output_images/test1/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/test1/output.jpg "Output"
[video1]: ./project_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

For each rubric point, I am using descriptions and supporting figures / images to explain how each rubric item was addressed, and which part of the code correspond to the rubric.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the Jupyter notebook located in "./P2.ipynb".

The code started by preparing `objpoints`, which will be the (x, y, z) coordinates of the chessboard corners in the world. The assumption here is that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Then `v2.findChessboardCorners()` was used to find corners (`imgpoints`) in the chessboard calibarion images. 

These `objpoints` and `imgpoints` are used to compute the camera matrix and distortion coefficients with the `cv2.calibrateCamera()` function.  I then applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

The code for defining each step in the pipeline is located in the 3rd code cell of the Jupyter notebook.

The code for using the pipeline and generating each step's image is contained in the 4th code cell of the Jupyter notebook.

#### 1. Provide an example of a distortion-corrected image.

For distortion correction, I used the `cv2.undistort()` function with the camera matrix and distortion coefficients from the camera calibration step. After I applied the distortion correction to one of the test images, it look like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the `generate_binary_image()` function of the 3rd code cell of the Jupyter notebook.

I used saturation, lightness, and x Sobel gradient thresholds to generate a binary image. I first found pixels that met the Sobel x thresholds, and then found the pixels satisfying both the saturation AND the lightness thresholds. These two sets of pixels were combined to form my binary image. Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the `warp()` function of the 3rd code cell of the Jupyter notebook. 

The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. It then used `cv2.getPerspectiveTransform()` to find the transformation matrix, and `cv2.warpPerspective()` to actually warped the image. In the pipeline I setup the source and destination points in the following manner:

```python
bottom_left = [200,720]
bottom_right = [1100,720]
top_right = [683,450]
top_left = [597,450]

dst_left_x = (top_left[0]+bottom_left[0])/2
dst_right_x = (top_right[0]+bottom_right[0])/2
dst_bottom_left = [dst_left_x,720]
dst_bottom_right = [dst_right_x,720]
dst_top_right = [dst_right_x,0]
dst_top_left = [dst_left_x,0]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 398, 720      | 
| 1100, 720     | 891, 720      |
| 683, 450      | 891, 0        |
| 597, 450      | 398, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the `find_lane_pixels()` and `fit_polynomial_combo()` function of the 3rd code cell of the Jupyter notebook.

I used histogram on the bottom half of the warped binary image to identify the x positions of the lane lines. Then I used sliding windows to find the lane pixels, adjusting the windows based on the average x positions of the lane pixels in a slide.

Then I used the lane pixels and pixel-to-meter conversion ratios to find polynomials coefficients (using `np.polyfit()`) for both the pixel space and meter space. The result is kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the `measure_curvature_real()` function of the 3rd code cell of the Jupyter notebook.

I used the polynomials to find the x positions of the lanes, and then calculated the radius of curvature at those positions. After finding the center of the lane from those x positions, the offset is calculated by comparing the center of the lane to the center of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the `generate_lane_overlay()` function of the 3rd code cell of the Jupyter notebook.

I made a mask with the lane pixels and lane region (defined by left and right lane line polynomials). I then warped the mask and put it on top of the the undistorted camera image. Texts for the radius of curvature and the center offset is drawn on the image as well.

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The code for this step is contained in the `process_image()` function of the 5th code cell of the Jupyter notebook.

For processing the video, I took several additional steps: 

1. A `Line()` class is defined to keep track of info generated in the pipeline. 
2. I defined the `search_around_poly()` function to find an image's lane pixels around a (previous) set of polynomial fits.
3. The pipeline alternated (or "reset") between `search_around_poly()` and `find_lane_pixels()` depending on how many frames had failed to find good polynomials (via a counter in `Line`). 
4. I counducted sanity check on the radius of curvature of the two lane lines, the distance between the lanes (from their x positions), and the parallel-ness of the lanes (by compare the slopes from the starting point to the end point of the polynomials). If the current frame passed the sanity check, info are updated in `Line`. Otherwise, info are kept until a better frame. Lane overlay are drawn with info from `Line`.

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One problem I faced is getting bad binary image and fitting bad polynomials. Some pixels were erroneously identified as lane pixels (e.g. shadows on the road appear strongly on the s-channel). Sometimes not enough lane pixels were found because thresholds were too strong for the road condition.

The pipeline might fail when the lighting condition "blends" the lane and the surrounding, when the color of the road is very light, and when other elements (substance on the road, reflection from cars) in the image have similar features as a lane. To improve this, I can combine more methods and refine the thresholds to generate better binary image, if I were to pursue this project further.  
