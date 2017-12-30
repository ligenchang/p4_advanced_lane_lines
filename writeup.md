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

[image0]: ./output_images/undistort_test.png "Undistorted"
[image1]: ./output_images/undistort.png "Undistorted"
[image3]: ./output_images/thresholded.png "Binary Example"
[image4]: ./output_images/points1.png "Warp Example"
[image5]: ./output_images/fitted.png "Fit Visual"
[image6]: ./output_images/result.jpg "result"
[video1]: ./project_video_submit.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third and fourth code cell of the IPython notebook located in "lane.ipynb".  


to calibrate the camera, I need the object points and image points. The object points represent the 3d points in real world space and image points represent the 2d points we saw in the picture. In the camera_cal folder, there are multiple calibration image for me to use. loop through these images, we can use cv2.findChessboardCorners function to get the corners of these chessboard taken from different angle. The returned corners will be our image points.
With the calculated image points and object points, we can calibrate our camera by using cv2.calibrateCamera function and it returns the camera matrix, distortion coefficients, rotation and translation vectors parameter. After that, I can call cv2.undistort function with camera matrix and distortion coefficients. It will return us the undistorted image. In the below example, the image is from the challenge video. You can clearly see that the edge of the bridge becomes straight in the undistorted image.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

As above stated, I firstly undistort the image.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at function pipeline in line.ipynb).  Here's an example of my output for this step.  The code is as follow:

```python
combined_binary[(r_binary_original == 1)| (l_binary == 1)| (b_binary == 1) |(sxbinary ==1 )  ] = 1
```

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform is performed before generating thresholded binary image and I find that doing perspective tranform first could reduce the image noise. the code for perspective is in pers_transform function. it takes one img parameter and return the warped image and the Minv value which is used for map the image back to normal image. I chose the hardcode the source and destination points in the following manner:


```python
src = np.float32(
[[(img_size[0] / 2) - 65, img_size[1] / 2 + 95],
[((img_size[0] / 6) - 10), img_size[1]-35],
[(img_size[0] * 5 / 6) + 60, img_size[1]-35],
[(img_size[0] / 2 + 65), img_size[1] / 2 + 95]])

dst = np.float32(
[[(img_size[0] / 4), 0],
[(img_size[0] / 4), img_size[1]],
[(img_size[0] * 3 / 4), img_size[1]],
[(img_size[0] * 3 / 4), 0]])
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:
In order to find the left line and right line, I need to get the histogram graph of the bottom half image. the peak of the histogram should be the start point of left line and right line. The I use a sliding window to search the left line and right line pixels by shifting the center of the window. We make a polyfit based on the found pixels for left line and right line. For the next frame, as we already know the fittef line for the last frame, we can use it as reference but will add one margin vallue ro re-search the pixels and polyfit to slightly adjust it to avoid blindly search every time. Sanity check will be preformed after each search and will reject the values if the cuvature changed to much or average x values changed a lot or the lines are not parallel.
The line for this implementation is in `line.ipynb` in function 'getfit(). Below is the example for identified lane-line pixels and fitted line:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius in my code in `line.ipynb` in function 'getfit():

```python
#calculate the curvature
y_eval = np.max(lefty)

#Define conversions in x and y from pixels space to meters
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

# Fit new polynomials to x,y in world space
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
# Calculate the new radii of curvature
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
# Now our radius of curvature is in meters
print(left_curverad, 'm', right_curverad, 'm')   
```

For the position related to center, I use the average value if the two fitted lines's start point and minus the image's width/2 and conver it to meters to get the car position. Here is the code:

```python
nowposition=((left_fitx[-1]+right_fitx[-1])/2-640)*3.7/700
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `line.ipynb` in the function `getfit()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)


Here's a [link to my video result](./project_video_submit.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1)For the project video, the failure usually happened on the shade area and the the road with high brightness. To overcome the failure, I did the histogram equalization to add the contrast and make the image more sharp and it really increased the robustness.
2)Though I find the s_binary could indentify the yellow color well, but it will bring the noise of high brightness point expecially the point is near the line
3)For the challenge video, the black line of the new road and the shade will be the big noise, to mitigat it, i only choose the r channel and s channel to make it only identify the yellow color and white color. I also perform histogram equalization twice to increase the contrast.
4)To make the video more smooth, i added one filter to do the sanity check. Most of the situations, it will use the existing average fitted line as the basic search for the next frame, but if it can't satisify the criteria, it will use the last averaged fit value.
