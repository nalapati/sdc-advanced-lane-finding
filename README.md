# Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

This project implements a software pipeline to identify the lane boundaries, the radius of curvature and offset of a car from the center of the lane in a video.

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
Camera Calibration is done in Cell 4, 21 in the ipython notebook in the project. The objective of camera calibration in this step is to correct for radial distortion, this is effectively results in mapping 3D points (virtual sphere surface) onto a 2D plane. In order to do this I used the supplied chessboard calibration images that contain 9x6 inner vertices to perform the calibration. Simply create a 3D array of points representing these 9x6 images and use the ``cv2.findChessboardCorners`` function to get the 2D image points. Collect the image points and object points on all calibration images and use the ``cv2.calibrateCamera`` to get the camera matrix and the distortion coefficients to undistort the images. ``cv2.undistort`` undistorts images.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/chessboard_undistort.jpg)

* Apply a distortion correction to raw images.
Undistortion is done in Cell 22 in the notebook, Applying the undistortion on an image from a car.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/straight_lines1_undistort.jpg)

* Use color transforms, gradients, etc., to create a thresholded binary image.
Thresholding is done in Cell 31 in the notebook, after tuning and experimentation the binary image is a combination of a thresholded image after applying sobel edge detection along the X direction with a kernel size of 7 and the red channel and using a thresolded saturation channel (converting BGR to HLS) and using a threshold of (90, 255)

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/thresholded_images.jpg)

* Apply a perspective transform to rectify binary image ("birds-eye view").
Perspective transform is done in Cell 24 in the notebook, the idea is to select 4 points in the image representing the region of interest that resemble a trapezium with the slant edges in line with the road lanes(need a calibration image with straight lines and the car dead center in the lane), and map them to 4 points in a birds eye view image. The transformation matrix and inverse transformation matrix are obtained using the ``cv2.getPerspectiveTransform`` method. The idea is to map every point in the source image to the destination image reducing the z distance of faraway lane points so they all appear to be at an equal distance from the camera giving the perception of a birds eye view.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/unwarped.jpg)

* Detect lane pixels and fit to find the lane boundary.
Lane line detection is done in Cell 25 in the notebook, here for every image independent of any other image I take a histogram of the lower half of the image to find peaks that represent the road lanes. (This is done for simplicity, an optimization is to use lane coordinates from previous frames in a stream). I expect to find two peaks in the histogram, one corresponding to each left and right road lane.
```
#
#  x1,y1         x2
#    *************-
#    *           * h
#    ****(x,y)****-
#   y2
#    |     w     |
```
Assuming the peak coordinates are (x,y) above, I run a sliding window along (x,y) from bottom of the image to the top of the image. In the window area I calculate the mean x coordinate of nonzero pixels to recenter the sliding window as it is moved from the bottom to the top of the image. All nonzero points aid in fitting a 2nd order polynomial (``np.polyfit``) along the left and the right lanes respectively.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/binary_image.jpg)

* Determine the curvature of the lane and vehicle position with respect to center. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Radius of curvature and offset from center and rendering the lane overlay are in Cells 9, 10, 26.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/lane_overlay.jpg)

* Pipeline
Code for the pipeline is in Cells 27, 28. The idea here is to run the stateless lane detection in Cell 25 to get a left and right lane line. Two key enhancements here are smoothing and skipping. In the smoothing step, the lane for the current frame is derived from the curve coefficients of the last 10 frames. In the skipping step, I check to see if the x coordinates of the left and right lanes are a fixed distance apart from each other (relatively fixed width), if not, just use the previous frame's lane lines.

![alt tag](https://raw.githubusercontent.com/nalapati/sdc-advanced-lane-finding/master/project_video_output.mp4)
