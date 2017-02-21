# SDCND-P4-Advanced-Lane-Finding
###In the context of traffic control, a lane is part of a roadway that is designated for use by a single line of vehicles, to control and guide drivers and reduce traffic conflicts [wikipedia](https://en.wikipedia.org/wiki/Lane). Lane finding is important for perception in order to know about the environment and the road. This project is applying computer vision techniques to detect the lane where the car is located for perception. 
---

**Advanced Lane Finding Project**

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Use color mask and sobel filters to create a thresholded binary image.
* Detect lane pixels and fit to find the lane boundary. There are two algoritms to perform this task:
i.	fit_polinomial_sliding_windows and ii.	fit_polinomial_based_on_lines 
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./images/corners.JPG "Corners"
[image2]: ./images/Undistorted_Image.JPG "Undistorted Image"
[image3]: ./images/Road_Undistorted_Image.jpg "Road Undistorted Image"
[image4]: ./images/birds_eye_view.JPG "birds eye view"
[image5]: ./images/color_mask_and_Sobel_Filters(mask).JPG "color mask and Sobel Filters"
[image6]: ./images/color_mask_and_Sobel_Filters.JPG "color mask and Sobel Filters"
[image7]: ./images/fit_polinomial_sliding_windows.JPG "fit_polinomial_sliding_windows"
[image8]: ./images/fit_polinomial_based_on_lines.JPG "fit_polinomial_based_on_lines"
[image9]: ./images/Lane_finding.JPG "Lane_finding"
[image10]: ./images/final_result.jpg "final_result"

[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Camera Calibration

####1. Camera calibration consist to find the quantities internal to the camera that affect the imaging process. The first step is case extracting object points and image points for camera calibration.

The code for this step is contained in the second code cell of the IPython notebook located in "./camera_calibration.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]
I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  The code for this step is contained in the last code cell of the IPython notebook located in "./camera_calibration.ipynb" 

I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image2]


###Read and undistort image

####1. read the image and apply the distortion coefficients found in the camera calibration, this help to reconstruct a world model with more accuracy.
The code for this step is contained in the function undistort_image() of the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb" 
![alt text][image3]

####2. perspective transform

The code for my perspective transform includes a function called `getPerspectiveTransform_values()`,  in the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb".  The `getPerspectiveTransform_values()` function takes as inputs an image (`image size`), as well as source (`src`) and offset .  After that it will apply the `warp_image()` function with the return of the perspective transform and  the image. 
To know the source (`src`) for this perspective transform I used the following code:
```
ltop = 0.07
lbottom = 0.44
x1, y1 = int(round(img_size[0]*(0.5-ltop))) , int(round(img_size[1]*(0.579166667 + ltop*0.847222222)))  
x2, y2 = int(round(img_size[0]*(0.5+ltop))) , y1
x3, y3 = img_size[0], img_size[1]
x4, y4 = 0, img_size[1]
        
src = np.float32([[x1,y1],[x2,y2],[x3,y3],[x4,y4]])

```

The values used: 0.579166667,  0.07 and 0.44 are the result of checking the image and get lineal functions to test it.

![alt text][image4]

####3.Use color mask and sobel filters to create a thresholded binary image: 
#####a.	Apply color mask:
######i.	Select yellow color: transform the RGB image to HSV image and use the following range: [22, 35, 80] to [100, 255, 255]
######ii.	Select white color: transform the RGB image to HLS image and use the following range: [0, 180, 160] to [150, 255, 255]
######iii.	Filter for saturation:  transform the RGB image to HLS image and use the following range: [20, 0, 100] to [100, 255, 255]
######iv.	Apply color mask using Or operator: mask[(hsv_yellow ==255)| (hls == 255) | (hls_white == 255)] = 1
#####b.	Apply Sobel filter: This is applied in order to detect edges and remove the pavement color with a color mask using HSV image format.
#####c.	Combine color mask and Sobel Filter: Apply “or” operator for both outputs. This is the result:
![alt text][image5]
The code for this step is contained in the second code cell of the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb" 
Here's an example of my output for this step.  (note: For this image I applied bitwise_and using as a mask the restult of this step)

![alt text][image6]


####4. Detect lane pixels and fit to find the lane boundary. There are two algoritms to perform this task:
#####a.	fit_polinomial_sliding_windows
Compute lines for the first frame or the last line found is very different to the previous lines (average):
######1.	Divide the image in 2 parts and perform a sum of the bottom part for axis “y” over axis “x” and get a histogram of the sum (axis X).
######2.	Get the maximum value for the left and right side, probably they are a good starting point for lane finding.
######3.	If there is a lot of values (200) greater than the middle of the max value, then reevaluate it only for the bottom window (1/9), probably it is a curve and we only need the starting point.
######4.	 Perform window sliding windows to know the area of interest, for the re-center next window on their mean position it also will affect the re-center for the new position in y:
win_y_left = int(win_y_left- (window_height-abs(leftx_current-old_leftx_current)))
######5.	Fit a second order polynomial function for each line
This is the result:
![alt text][image7]
The code for this step is contained in the function `fit_polinomial_sliding_windows` of the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb" 
#####b.	fit_polinomial_based_on_lines - Compute the lines based on previous lines:
######1.	Use the last polynomial found and add margin to know the area of interest.
######2.	Fit a second order polynomial function for each line
######3.	It is most faster, but if the changes are big regarding the last line found it will not be useful and we will go back to use fit_polinomial_sliding_windows
This is the result:
![alt text][image8]
The code for this step is contained in the function `fit_polinomial_based_on_lines` of the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb" 

####5.	Lane finding
####a.	With the previous techniques we found the second order polynomial function for each line, then we know the shape of the lane. 
####b.	Draw the 2 lines
####c.	Calculate the radius of curvature of each line with the respective scale, in this case y= 3/80 (3 meters for 80 pixels) and x= 3.7/740 (3.4 meters for 740 pixels). For the final curvature I performed an average over both radius  of curvature (left line and right line) 
####d. Calculate the position of the vehicle with respect to center. It is a average of the bottom pixels (x value) of each  line found:
`int((left_fitx[-1]+right_fitx[-1])/2 - (img.shape[1]/2))`
The code for this step  is contained in the function `draw_lines` of the IPython notebook located in "./P4-CarND-Advanced-Lane-Lines-master.ipynb" 

This is the result:

![alt text][image9]
####6. Example image of your result plotted back down onto the road such that the lane area is identified clearly. Final result:

![alt text][image10]


---

###Pipeline (video)

Here's a [link to video ouput result](./project_video_output.mp4)
Here's a [link to challenge video result](./challenge_video_output.mp4)
Here's a [link to harder challenge video result](./challenge_video_output.mp4) Note: This is only a test that there are a lot of steps that can be improve.

---

###Discussion

#####1. On this project there is a lot of combinations for color mask and Sobel filter, or even other techniques. A little change can have a huge impact on the result. 
#####2. To get a better result for more curved lines I checked a lot of approaches for the `fit_polinomial_sliding_windows` and my solution was to reevaluate the histogram and move in 'y' is inversely proportional to move in 'x'.
#####3. My algorithm is not good enough for the harder challenge, but I can find some parts of the video (video attached)

the order that I took to perform the tasks:
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Use color mask and sobel filters to create a thresholder binary image.
Is because the mask will be clean to analysis, First I tried on the contrary and I was getting more noise.

For the hard frames to work on, I'm saving the last 5 lines in the `Lines` class to compare to the new line found. To know if this is good I'm checking it with a Mean squared error <40 and the evaluation af all the frame in the function `fit_polinomial_sliding_windows`

