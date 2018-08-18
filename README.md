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

The code for this step is contained in the third cells of the Jupyter notebook Advanced_Lane_Lines.ipynb.

The OpenCV functions **findChessboardCorners and calibrateCamera** were used for image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprised the input. Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by **findChessboardCorners**, are fed to **calibrateCamera** which returned camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera (and lens). 
The cal images produced the following vector of **Distortion Coeff** (k1,k2,p1,p2,k3)

(-2.41018767e-01 ,-5.30665431e-02 ,-1.15811319e-03 ,-1.28284935e-04, 2.67026162e-02)
          
          Where, k1,k2,k3 are radial distrotion coefficients
                                    
                                    and p1,p2 are tangential distortion coefficients

The **camera coeffs** are as follows:

![Camera Coeffs](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/camera%20coeffs.PNG)


The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function **drawChessboardCorners**:

The original image and the transformed image is shown here:
![before-after comparison](/images/undistort_output.png)

### Apply color transforms, gradient to create a threshold binary image

Next the RGB image was traslated in  HLS color space to obtain a binary thresholded image:

![color transform](/images/color_transform.png)

### Perspective transform ("birds-eye view"). 
The vertices coordinates were used to perform a perspective transform. The polygon with these vertices is drawn on the image for visualization. Destination points are chosen such that straight lanes appear more or less parallel in the transformed image.


![Perspective](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/perspective_transform.png)

### Detect lane pixels (sliding window search). 

Using histogram, the most likely position of the lanes were found out. The approach was to compute a histogram of the bottom half of the image to find the bottom-most x position (or "base") of the left and right lane lines.

![Histogram](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/histogram.PNG)

Then a sliding window search, starting with the most likely positions from the histogram was used to identify the lane markers for ahead portion of the road. A set of 10 windows of width 100 pixels were used for sliding window approach. The x & y coordinates of non zeros pixels were found, then a polynomial curve fit for these coordinates and the lane lines are drawn.

Sliding Windows Search for Lane markers.  
![Sliding window transform](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/sliding_window.png)

### Searching around previously detected lane lines 

Since consecutive frames are likely to have lane lines in roughly similar positions, the search was done around a margin of 50 pixels of the previously detected lane lines.

Sliding Windows search for Lane markers. 

![Sliding window transform](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/sliding_window1.png)

### Computing Radius of Curvature

The radius of curvature is based upon this [website](https://www.intmath.com/applications-differentiation/8-radius-curvature.php) and calculated using this formula (represented in simplified form)

     curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

The position of the vehicle with respect to the center of the lane was calculated with the following lines of code:

      lane_center_position = (r_fit_x_int + l_fit_x_int) /2
          center_dist = (car_position - lane_center_position) * x_meters_per_pix

r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).


### Inverse transform and output For the final image we:

Next the following three taskes were carried out
- Paint the lane area
- Perform an inverse perspective transform
- Combine the processed image with the original image.

 A polygon was generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results of the draw_lane function:

![final image](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/final_image.png)

### Example Result We apply the pipeline to a test image. 
The original image and the processed image are shown side by side.

![radius image](https://github.com/soumende1/AdvancedLaneFinding/blob/master/images/image_radius.png)

### Fine tuning the model : The Final Video


There were some frames where no lanes could be detected or the lanes  did not make sense. This gave rise to "bad frames".
The bad frames, were detected when the following conditions are met:

- No pixels were detected using the sliding window search or search around the previously detected line.
- The average gap between the lanes is less than 0.7 times or greater than 1.3 times the globally maintained moving average of the lane   gap.

Then the Averaging lanes were carried out: The lane for each frame is a simple average of 12 previously computed lanes using the get_averaged_line method


  All the above pipeline techniques were used to create the output 'video' file "AdvancedLaneDetection_video.mp4"  using the 
    pipeline_final method.
    
    These files  uploaded in GitHub repository and the representative gif file is shown below

![final Video](/images/project_video_output.gif)

### The Challenge Video



## Problems or Issues Faced

Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

The challenges I faced and future improvement opportunities can be categorized in to the following buckets:

### Gradient & Color Thresholding
The problems I encountered were almost primarily due to lighting conditions, shadows, discoloration, etc. It wasn't difficult to dial in threshold parameters to get the pipeline to perform well on the original project video I had to experiment a lot with gradient and color channel thresholding. The lanes lines in the challenge video was  difficult to detect as they were either too bright or too dull. This prompted me to have R & G channel thresholding and L channel thresholding. Tuning the threshholds for each transform is time consuming work. One setting may work for one picture but not work for other pictures. 

### Bad Frames
The challenge video has a section where the car goes underneath a tunnel and no lanes are detected. To tackle this I had to resort to averaging over the previous well detected frames. The lanes in the challenge video change in color, shape and direction. I had to experiment with color threholds to tackle this. Ultimately I had to make use of R, G channels and L channel thresholds.

### Points of failure & Areas of Improvement
The pipeline seems to fail for the harder challenge video. This video has sharper turns and at very short intervals.I think what I could improve is. to tackle this in future we can explore taking  a better perspective transform. We can also explore taking the average over a smaller number of frames. Right now I am averaging over 12 frames. This fails for the harder challenge video since the shape and direction of lanes changes quite fast.




