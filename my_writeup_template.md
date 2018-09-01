# **Advanced Lane Finding**

---

###Advanced Lane Finding Project

**The goals / steps of this project are the following:**

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use colour transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to centre.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/distortion_corrected_image.png "Corrected Chessboard"
[image2]: ./camera_cal/calibration4.jpg "Not suitable chessboard"
[image3]: ./camera_cal/calibration10.jpg "Suitable chessboard"
[image4]: ./output_images/distortion_corrected_testimages_6.png "Calibrated Image"
[image5]: ./output_images/test_image_correct_warp.png "Warped Chessboard"
[image6]: ./output_images/warped_corrected_testimages_8.png "Warp Example"
[image7]: ./output_images/binary_testimages_7.png "Binary Example"
[image8]: ./output_images/histogram_output.png "Histogram of lanes"
[image9]: ./output_images/sliding_window.png "Sliding window"
[image10]: ./output_images/previous_search_output.png "Previous Search Method"
[image11]: ./output_images/annotated_lane.png "Annotated Lane"
[video1]: ./output_project_video.mp4 "Standard Video"
[video2]: ./output_challenge_video.mp4 "Challenge Video"

---

### Files Submitted & Code Quality

#### 1. The project includes the following files:
* [CameraCalibration.ipynb](./CameraCalibration.ipynb) - notebook of camera calibration process.
* [camera_calibration.p](./camera_calibration.p) - pickled file containing camera calibration data
* [SingleImagePipeLine.ipynb](./SingleImagePipeLine.ipynb) - notebook of single image pipeline 
* [Colour_Investigations.ipynb](./Colour_Investigations.ipynb) - notebook of image processing investigations 
* [VideoPipeline.ipynb](./VideoPipeline.ipynb) - notebook of pipeline to make annotated video.
* [output_project_video.mp4](./output_project_video.mp4) - video of annotated lane on project_video.mp4
* [output_challenge_video.mp4](./output_challenge_video.mp4) - video of annotated lane on challenge_video.mp4
* [writeup report](./my_writeup_template.md) - markdown file summarising the results



### Camera Calibration

#### 1. Computing the camera matrix and distortion coefficients. 

The code for this step is contained in the jupyter notebook: [CameraCalibration.ipynb](./CameraCalibration.ipynb).  

 The code to run the camera calibration is contained in cell [7].  

 **The first part utilises the function "chessboard_points"**

 * This function takes in 2 parameters: the number of horizontal squares and number of vertical squares of the chessboard images to be used for calibration. 
 * The function will cycle through all of the chessboard images in a folder location, detecting the inner corners positions of the chessboard and storing them into an array. It also creates another array the same size which will be used of 3D points. I added a function from opencv that can increase the accuracy of the corner detection in an image 'cv2.cornerSubPix'.
 * The function will return an array of 2D points, array of 3D points of the same size, plus the size of the image.
 * The function will also report how many of the images in the folder were able to be used for calibration. In this camera calibration only 17 out of 20 images were able to be used.
 
The left image shows an image which couldn't be used as it couldn't find 9 squares horizontally due to the corner of the image not visible. The right image could corners could be detected as all of the squares are visible.

![alt text][image2]{: width="300px"}
![alt text][image3]{: width="300px"}

**The second part utilises the function "generate_calibration"**  

* This function takes the parameters output from chessboard_points, and using the opencv function "cv2.calibrateCamera"; it will return a camera matrix and distortion parameters that can be used to correct any image taken by that camera.
* The function takes these 2 parameters, labels them mtx and dist and stores them into a pickled file called: [camera_calibration.p](./camera_calibration.p)

**To validate/use the camera calibration**

I created 2 functions to correct an image:  
**get_calibration(): cell [5]** - This function reads in a pickled calibration file and returns the camera matrix and distortion parameters into memory.  
**image_correct(): cell [6]** - This function takes in an image, calls the get_calibration function and utilises the opencv function 'cv2.undistort' to return a corrected image

The code in cell [9] reads in an image, calls the 'image_correct' function to obtain a corrected image.
Using the opencv function 'cv2.drawChessboardCorners' the detected corners are drawn onto the source image. This is shown on the original image in the left. The right side shows a corrected image. The horizontal lines appear straight in the corrected image.

![alt text][image1]

### Pipeline (single images)

#### 1. An example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images:  

![alt text][image4]

Understanding the camera corrections in this image is not easy to understand. The yellow sign on the left side of the image can show the correction, as in the original image it looks slightly bent compared with the straight line more clear sign in the calibrated image. The vehicle hood is also an easy difference to recognise the change in the image.


#### 2. Perspective transform.

The code for this step is contained in the jupyter notebook: [SingleImagePipeLine.ipynb](./SingleImagePipeLine.ipynb). 

The functions: get_coordinates(), perspective_transform() and corners_unwarp() and are contained in cells [6], [7] and [8] of the notebook.  

The steps of this pipeline are: 

* The corners_unwarp() takes in an image.
* Inside the function the camera calibration is obtained from the pickled file: line 6 of cell [ 8 ]
* The image is processed with image_correct() function to get a calibrated image: line 8 of cell [ 8 ].  
* The next step uses a function called 'chessboard_corners'; this function is used for this experiment to obtain the 4 inner corners in order to draw a rectangle shape to understand the effects of calibration and warping : line 11 of cell [ 8 ].   
* With the 4 key points of interest - in this case the 4 corners of the rectangle; the source points for the warp are loaded into an array: : line 15 of cell [ 8 ]. A function get_coordinates() is used to extract the x,y data of each coordinate: line 19 of cell [ 8 ]. 
* An array with 4 destination points is created from the 4 corners of the detected corners: line 22 of cell [ 8 ]. 
* The function perspective_transform() takes the destination points, source points and the image: line 47 of cell [ 8 ]. This function takes the source and destination points and applies an openCV function 'cv2.Perspective' to return a transform matrix. It also returns an inverse transform matrix too. In order to get a warped image; another openCV function, 'cv2.warpPerspective' takes the passed in image and the transform matrix to produce a corrected image: line 12 of cell [ 7 ].
* An image is return that is corrected to a top down view.

The images below show the processing of the image:  
The calibrated image was used to detect the corners and set the base rectangle - shown in cyan. These points formed the source for the warp transform.
I overlaid this rectangle onto the original image which allowed visualisation of the original image compared with the camera calibration output.
After applying the warp pipeline as described above the 3rd image result is obtained. This is a flattened top down view by applying the transform matrix. The dark blue rectangle shows the destination points. It can be seen that the corners are transforming between the perspective calibrated image to the warped image and align up with the points in both cases.

![alt text][image5]

The function birds_eye_view() in cell [10] is the transform used for road lane detection. The source points are an area which represents the road ahead in an image. The image passed in is corrected and the transformed to a top down view using these coordinates. The flattened image gives a top down view of the road ahead. The images below show the before and after of this transform:

![alt text][image6] 


#### 3. Create a thresholded binary image.

I explored many colour systems transforms and gradient applications to the test images. This exploration can be viewed in the notebook: [Colour_Investigations.ipynb](./Colour_Investigations.ipynb). 
I used this notebook to test different colour modes on the driving scene test images and trialled different parameters to see which could provide the best results for lane lines detection.

The final selected methods were applied in notebook: [SingleImagePipeLine.ipynb](./SingleImagePipeLine.ipynb)

The main function 'image_conversion_pipeline()' which creates the threshold image is in cell [14] of the notebook. I used a combination of colour and gradient thresholds to generate the binary image - the thresholds are passed into this function. Finally I apply a mask over the area to only focus on the region of interest.

The image below;  
Left: Original image
Centre: Image after thresholding applied and mask mask applied
Right: Perspective transform of the source area

![alt text][image7]

The image shown is a straight lane image.
The red dashed line is simply overlaid to allow a visualisation of how straight the perspective output is. Using this red line method to measure the straightness helped to tune the destination and source points for the perspective transform.


#### 4. Identifying and fitting lane line pixels.

The following process was used to identify the lane lines from the binary pixel image:  

* The function histogram_split() creates a histogram of the image - this highlights where the highest intensity of white pixels are on the bottom of the image. Getting this position can give a starting point for left and right lines.
* The Histogram shows two large spikes - these are the like likely positions of the right and left lines. The function splits into left and right and returns the two x positions of the image.

In this image the histogram returned the line points as: Left: 493, Right: 838

![alt text][image8]  

* In order to generate a line the full height of the  screen a sliding window approach is applied. Cells [18] & [19] take in the binary image and the left and right positions from the histogram. I set parameters to generate 9 windows for each line to be detected, as the image is 720 pixels high - this sets a window height of 80 pixels. The margin was set to 60 which sets the window width to 120 total; 60 either side of the centre point. The windows work their way up the image adjusting their centre point to best fit the pixels. An array of points are stored
* Using numpy function polyfit() a second order polynomial is generated from the array of points to represent each line. This is calculated in cell [20].

The image below shows a visual output:  

* The green windows that are the result of the sliding window process.
* The left(red pixels) and right(blue pixels) that exist inside of the windowed area to be used for the polyfit function
* The Yellow line is the application of the 2nd order polynomial being applied to a range of y-values spanning the height of the image.
* The result is two lines that can represent the lane lines.

![alt text][image9]  

* If this was a moving image then repeating the sliding window approach for every image would be slow. As the top most position is known from a the window approach, a search method was applied within a 'search_margin'.
* This can be seen in cell [22], this search will return an array of pixels within the specified margin.
* The numpy polyfit is again applied to this array to generate a 2nd order polynomial that can be seen in cell [23].

The image below shows a visual output:

This is very similar to the sliding window output, but instead of using the windows to search for pixels a green margin area is identified which is the area used to pick pixels to generate the polynomial.

![alt text][image10]


#### 5. Radius of curve and vehicle position.

In order to calculate the curvature of the visible road ahead:

Firstly was to establish a scale which the pixels could be related to.
The road lines in the US are defined as a 3.7m lane, and 3m:7m (line:dash).
This meant a scale could be established from the known width the lane should be and the detections that had been made.  

The code for the following can be seen in cell [25]

* I took the left and right points detected at the base of the image and subtracted their pixel values - this provided a lane width in pixels. 
* Knowing the actual lane should be 3.7m, then dividing by lane width in pixels gave the image an x scale of metres per pixel. 
* This process was repeated for the y axis, using 30 metres as the length of the visible road ahead.
* Using the equation to calculate the radius of curvature the real world curvature of the lane line can be calculated. Line 46 and 47 calculate the left and right lane line curvatures.

The results of the lane curvature for the image currently being processed is:

```
Left Curvature Radius in pixels: 4062.0p
Left Curvature Radius in metres: 677.3m
Right Curvature Radius in pixels: 4498.5p
Right Curvature Radius in metres: 749.9m
```

To calculate the position of the car with respect to the lane; the code for this can be seen in lines 30 to 35.

* This code finds the centre of the camera position between the two lane lines.
* Calculates the difference between this position and centre of the actual image.
* If the difference is negative then it determines the position of the car is left of centre, if positive then it is right of centre.

#### 6. Example image of your result plotted onto the road to identify lane.

To visualise the lane lines onto the image a function 'draw_lane_lines()' is utilised. This can be seen in code cell [26].

* The function creates lines from the left and right polynomials.
* It passed these points into a numpy array to be create an area to represent each line and inner lane.
* The lines and inner lane are then filled using openCV 'fillPoly()' and 'polylines()' functions.
* The left line is coloured red, the inner lane green and the right line is coloured blue.
* Annotation is applied to the image to show the left lane curvature of the road ahead.
* Also the position of the centre of the vehicle in relation to the centre of the land is also added to the image.

An example output of the image with the lane lines drawn is shown below:

![alt text][image11]

---

### Pipeline (video)

The code for this step is contained in the jupyter notebook: [VideoPipeline.ipynb](./VideoPipeline.ipynb).  

#### 1. Final Video Output.

This is the final output video: [Standard Video Result](./output_project_video.mp4)

The lane line detection is fairly stable and manages to keep track of the lane all the way through the video.

I applied my pipeline to the challenge video. The output of this can be viewed here: [Challenge Video Result](./output_challenge_video.mp4)

---

### Discussion

#### 1. Discussion of problems with my implementation of this project.  

The standard video appears ok as the lanes are tracked and it makes it all the way to the end.

I spent a lot of time trying to find the best way to detect the lane lines to try to get an optimum detection result. However I did not spend an equal amount of time on the tracking part of the lines.

This can be clearly seen in the challenge video. The detection falls over very quickly as it is following the lines. The lane line moves into a shadow area with a two tone road surface, this cause the detection many issues and the lane lines are twisted, and can't recover.

To improve this I could have implemented some logic to maintain an approximate parallel distance between the lines at all times. Additionally if each line was measured for variance to the last line I could have restricted the amount of variation a line could move which would have smoothed out the line generation and maybe maintained a more stable output.
