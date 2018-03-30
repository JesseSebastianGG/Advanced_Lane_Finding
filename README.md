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

[image1]: ./illustrations/undist_calib.jpg "Calibration Undistorted"
[image2]: ./illustrations/dist_warp.jpg "Undistorted and Warped"
[image3]: ./illustrations/threshold.jpg "Combined color and gradient"
[image4]: ./illustrations/pipeline.jpg "Binary warped outputs"
[image5]: ./illustrations/find_lines.jpg "Fit Visual"
[image6]: ./illustrations/annotated.jpg "Output"
[video1]: ./project_video_solution1.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Code Navigation

Note that the jupyter notebook starts with an index, whose 11 sections steer the reader through relevant areas of design and the project rubric. The code itself is sectioned correspondingly. I reproduce it here:

            Index 
            1. Camera calibration
            2. Correct for image distortion (using matrix and coef's from step 1)
            3. Color and gradient threshold
            4. Warp image (perspective transform)
            5. Combine 1-4 ("pipeline")
            6. Find lanes
            7. Calculate curvature and offset
            8. Draw lanes
            9. Define class for tracking lane properties
            10. Consolidate 1-10 ("process")
            11. Run on videos


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in section 1 of the notebook. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one. Note that the second image demonstrates undistortion, combined with gray transform; the remining two show the warpping process back and forth (see below).
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

See section 3 of the notebook for code. I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step, applied to the provided test images. In the third column, contributions are stacked: green is gradient and blue is color.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

See section 4 of the notebook for code.

The code for my perspective transform includes a function called `warp()`.  The `warper()` function takes as inputs an image (`img`), as well as the Boolean 'inverse'; for inverting a warp, set inverse=True.  I chose the hardcode the source and destination points in the following manner:

```python
    # Source (zooming in on appropriately distant sections of road, thanks to qt)
    src = np.float32(
    [[673,444],
    [1059,691],
    [248,691],
    [605,444]])
    # Destination (eyeballing central rectangle)
    dst = np.float32(
    [[1005-25,20], # these +-25s were tuned by hand using all test images
    [1005,700], # a large y-coordinate helped the inverse matrix extend the lane down to the bonnet
    [295,700],
    [295+25,20]]) 
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto all test images and its warped counterpart to verify that the lines (curves) appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

See section 6 of the notebook for code.

There are two essentially similar ways of finding lane lines, depending on whether prior frames' lines are known. Both methods involve sliding-windows-like scanning, with the blind search searching the entire half, and the posterior search starting within a margin of the previous lines. Since both functions take a warped binary image (output of the above transformations and filters), almost all the information pertains to lane lines; the histogram approach enables the strongest signals in each half to be chosen as lane lines, initially. Once arrays of pixels were gathered, the numpy polyfit method was applied to fit each lane line to a second-order polynomial. Order 2 is a compromise between capturing curvature (any lower wouldn't) and avoiding over-fitting (roads are designed to be locally 'smooth', with negligible higher-order derivatives locally).

These images illustrate an initial image having its lane lines fit, followed by a second image relying on the first image's data as prior information.



![alt text][image5]



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

See section 7 of the notebook for code. See also [this link](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) for maths.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_solution1.mp4)


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline fails for the challenge video. It finds incorrect lanes persistently.

I added print statements to the process() function in order to see when previous lines were being used. For the standard video (which my solution passes), all but the first image are processed using the previous one. This is not surprising since the images are very smooth.

However in the challenge video, almost all frames are ignored for subsequent ones, suggesting my thresholding is too harsh. It is not so obvious however, since the lanes found initially are incorrect, so I wouldn't want to encourage the code to give weight to previous fits if they are wrong.

Given more time I would address this, perhaps by using probability to give weight to lines of average gradient in both left and right halves of the image. This might help avoid using cracks in the road (near vertical) or shadows on the side (near 45 degrees).
