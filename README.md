## Advanced Lane Finding
![[Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./output_images/0.jpg)

In this project,I wrote a software pipeline to identify the lane boundaries in a video.

The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[image1]: ../output_images/undistorted.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/color.png "Color Channel Extraction"
[image6]: ./output_images/sobel.png "Sobel Operation"
[image7]: ./output_images/combined.png "Everything Combined"
[image8]: ./output_images/histogram.png "Histogram"
[image9]: ./output_images/window.png "Sliding Window Search"
[image10]: ./output_images/Lane_detection.png "Lane detection"
[image11]: ./output_images/unwrap.png "Unwraping images"
[image12]: ./output_images/radius.png "Final Output"
[video1]: ./project_video.mp4 "Video"


The code is contained in the first code cell of the IPython notebook Advanced-Lane-Lines-Finder.ipynb. 

#### Step 1. Finding the Chessboard Corners

I started out by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.

After reading the images and grayscaling the images, so that the Chessboard image can be detected more easily,I used `cv2.findChessboardCorners()` to find the corners of the Chessboard. It returns two values; ret, which tells if the function found any corner in the given grayscaled image or not, and, corners, which contains the actual corner points of the chessboard.  

If the corners are found, then `objp`, which is just a replicated array of coordinates, will be appended to `objpoints` with a copy of it, every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

#### Step 2. Calculating Undistortion Parameters

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

#### Step 3. Undistorting Images

I then created a function `undistortImage()` which will take the output `mtx` and `dst` obtained from `cv2.calibrateCamera()` function to distort the images using `cv2.distort()` function.

The test images are then applied the `undistortImage()` function to compare the test images of distorted and undistorted images.

![alt text][image2]

#### Step 4 and 5. Defining Region of Interest and Wraping the images

The source points `src` and destination points `dst` are defined with the `src` being a trapezium and `dst` being a rectangle. ROI is defined under the trapezium region. And `WarpPerspective()` function performs the perspective transformation of the image and provides a bird eye view of the road. The wraped images are appened with `wrapedImages` list.

![alt text][image3]
![alt text][image4]

#### Step 6. Extracting Color Channel

Here, I defined `ExtractChannel()` function which takes in `image`, `colorspace`, `threshold` and `channel=0` input to extract the color `channel` of the given `colorspace` converted from the image between the minimum and maximum `threshold` values. After that, I tested on different values to get a sense of which of the following color channel gives the best output for detecting lane lines. Here I found that I can use the Saturation(S), Lightness(L) and Luma(Y) component of the images to detect the images.

![alt text][image5]

#### Step 7. Applying the Sobel operator to the images.

I defined a `Sobel()` function which calculates different types of Sobel operations on the images depending on the `sobelType` given to the function. I then tested for all the Sobel operations namely `Sobel X` , `Sobel Y`, `Sobel Magnitude` and `Sobel Directions`, for which I found that the `Sobel X` operation will work best for the lane detection.

![alt text][image6]

#### Step 8. Combining Everything

All the pervious operations are performed in a single function `combineEverything()` which chooses Saturation(S), Lightness(L) and Luma(Y) component of the wraped images and Sobel X operation to generate a binary image. The function is applied ti the test images, to check the accuracy of the detected lanes.

![alt text][image7]

#### Step 9. Plotting Histogram

A histogram is plotted which shows the sum of activated regions of the bottom half the combined images.

![alt text][image8]

#### Step 10. Finding Lines

The Sliding Window Search which firsts splits the histogram into two, left and right halves. It then draws the boxes and find if the non-zero pixels within the window. If the non-zero pixels within the window of left and right halves are greater than the minimum allowed pixels, then the next window is recenterd to their mean position. Then all the left and right lane indices are concatenated using `np.concatenate()`. All left and right line pixel positions were extracted and these were used to find the polynomial cofficients to fit the left and right halves of the histograms. The sliding windows search is plotted on the wraped images. 

![alt text][image9]

The `VisualizeLaneDetection()` function draws continous sliding windows to more accurately detect the lane lines.

![alt text][image10]

#### Step 11.Unwraping the Lanes onto the images

The `DrawLine()` functions wraps the binary image, detects the lane lines and, draws the lane onto the warped blank image. The warped blank image is then unwraped and warps the blank back to original image space using inverse perspective matrix. The returned image is then unwraped and stacked over the original image. The test images are tested to check if the lanes are detected are correct or not. 

![alt text][image11]

#### Step 12. Calculating Radius and Distance

The `CalculateRadiusOfCurvature()` function calculates the Radius of Curvature using the formula <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/997befd7b50afdfd71a4d018f4c4be7376d1f6bd" />. The Radius of curvature is then converted to meters using pixel per meter with the values
`    
    ym_per_pix = 30/720 
    xm_per_pix = 3.7/700 
`
#### Step 13. Defining the Pipeline

All the above functions are merged into a single function `lane_tracing()` and the pipeline is created

#### Step 14. The pipeline are ran on the test images

Here are some of the results from the test images

![alt text][image12]

#### Step 15. The pipeline is applied to the final project video

![alt text][video1]


### Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

The pipeline would fail if it detects steep turns and lots of curvature. It failed miserably in the harder video challenge( The video is present in the other branch). It will also fail if it detects the line created the newly created road and the pipeline will falsely detect the line as the lane image. 

To make it more robust and stop the flickering of lane lines, we can average out the points from the previous frames to have a smooth transition per frame. Also we can also introduce the deep learning methods to make the pipleline more accurate.
