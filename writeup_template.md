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

Project Repo: 
https://github.com/bbhulani/sdc_p4_adv_lane_lines.git


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

This code is implemented in camera_cal() and cal_undistort()

./out_images/cal_undist_chessboard.png

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image. 

Example of distortion corercted test image:
./out_images/distortion_corrected_test.png

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image color_gradient().  
I used the HLS colorspace and specifically the S_channel for threshold color transform  
**** I need more time to explose HSV space and maybe combine another threshold to solve the issue when there are shadows in the image. 
I used the sobel x operator for thrshold x gradient transform. 
I combined the 2 binary threshold transforms for the output image at this stage. 

Same image above after color and gradient transform:
./out_images/color_gradient.png

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Perspective transform is done in corners_unwarp()

The function takes in an undistorted image as input. The src and destination pts are:
    # source points
    offset1 = 300
    offset2 = 55
    offset3 = 100
    offset4 = 200
    src = np.float32([
    [offset1, img_size[1]],
    [(img_size[0] / 2) - offset2, img_size[1] / 2 + offset3],
    [(img_size[0] / 2) + offset2, img_size[1] / 2 + offset3],
    [img_size[0] - offset4, img_size[1]]
    ])
      
    #destination points
    dst = np.float32([
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] / 4), 0],
    [(img_size[0] * 3 / 4), 0],
    [(img_size[0] * 3 / 4), img_size[1]]
    ])
    
    I chose to use offsets from the midpoint of the X and Y axis lenghts for src points. 
    *** This could be further refined to improve land detection for challenge video. Need more time to do that
    
Same image above after perspeftive transform:
./output_images/perspective_transform

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I used the sliding window approrach to identify the lane lines in the perspective transformed image. Once the left lane and right lane pixels are identified I fit the pixels to a second order polynomial using np.polyfit. The left and right lane polynomials are smoothed so as to avoid transient changes in lane detection. 

Example output of lane identification on the perspective trarnsform image:
./output_images/sliding_window_test6.png

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature is computed in sliding_window() (towards the end of the function!)
The position of the vehicle respect to center is not computed ***

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In process_image() I warp the detected lane to original image space using inverse perspective matrix (Minv). I then stack that on the undistorted image as show in example below
./output_images/lane_plot_test6.png

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

./project_video_out.mp4
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Things that can be improved
1. When the lane has shadows from trees or passing under the bridge in the challenge video my code doesnt identify the lane correctly. This can possibly improved by using better color or gradient transforms
2. In the challenge video the lane being so close to the the divider causes the lane detection to get messed up. The perspective transforms src/dst area needs to be reworked to make it better
