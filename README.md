# Project Description - Advanced Lane Finding
This is the writeup for fourth project (Advanced Lane finding)

The basic objective of this project is to:

- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position with respect to center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position

### Compute the camera calibration matrix and apply distortion correction

The code for this step is contained in the first two code cells of the Jupyter notebook project.ipynb.

The OpenCV functions findChessboardCorners and calibrateCamera are used for image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by findChessboardCorners, are fed to calibrateCamera which returns camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera (and lens). The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function drawChessboardCorners:

The original image and the transformed image is shown here:
![before-after comparison](/images/undistort_output.png)

### Apply color transforms, gradient to create a threshold binary image

Next the RGB image was traslated in  HLS color space to obtain a binary thresholded image:

![color transform](/images/color_transform.png)

### Perspective transform ("birds-eye view"). 
The vertices coordinates were used to perform a perspective transform. The polygon with these vertices is drawn on the image for visualization. Destination points are chosen such that straight lanes appear more or less parallel in the transformed image.


![Perspective](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/perspective_transform.png)

### Detect lane pixels (sliding window search). 
Using histogram the most likely position of the lanes were found out. Then a sliding window search, starting with the likely positions from the histogram was used to identify the lane markers for ahead portion of the road. 10 windows of width 100 pixels were used for sliding window approach. The x & y coordinates of non zeros pixels were found, then a polynomial curve fit for these coordinates and the lane lines are drawn.

Sliding Windows Search for Lane markers.  
![Sliding window transform](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/sliding_window.png)

### Searching around previosly detected lane line 

Since consecutive frames are likely to have lane lines in roughly similar positions, we search around a margin of 50 pixels of the previously detected lane lines.

Sliding Windows Search for Lane markers. 

![Sliding window transform](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/sliding_window1.png)

### Inverse transform and output For the final image we:

Paint the lane area
Perform an inverse perspective transform
Combine the processed image with the original image.

![final image](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/final_image.png)

### Example Result We apply the pipeline to a test image. 
The original image and the processed image are shown side by side.

![radius image](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/image_radius.png)



