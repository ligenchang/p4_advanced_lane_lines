## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  


[image0]: ./output_images/histogram_equalize.png "histogram"
[image1]: ./output_images/undistort.png "Undistorted"
[image3]: ./output_images/thresholded.png "Binary Example"
[image4]: ./output_images/points1.png "Warp Example"
[image5]: ./output_images/fitted.png "Fit Visual"
[image6]: ./output_images/result.jpg "result"
[video1]: ./project_video_submit.mp4 "Video"

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


Introduction
---
This project is related to computer vision to correctly recognized the road lines in more complex situations and it needs mixed knowledges to make it work. We need to do the magic camera calibration and undistort the image adn then do the perspective transform to display the image in bird eye view. Then we need to get one thsesholded binary image to  detect the pixels and fit the line and do the saity check and adjustment.


Step1
---
If you observe the video carefully, you may find that in some areas, it's very bright and some areas, it's with tree shades. I tried to perform the histogram equalization to increase the contrast before other steps, here is the image before and after the histogram equalization.

![alt text][image0]

Step2
---
Next step is to do the perspective transform. In the class, it prformed the perspective transform after the thresholded image, but i find that it will increase more noises. So I suggested to do the perspetive transform first in step2. If we want to know the road situations in an bird's eye view, we need to tranform our car camera image. To achieve that, we need to find the source points and destinations points. To validate the points selection is correct, we can check if the line is parallel roughly in tranformed image. As the camera image has a car itslef in the bottom so that we need to trim that part to remove the noise. Here is the image for original image and tranformed image:
![alt text][image4]

Step3
---
Now we have the bird-eye view image. In order to detect lines, we need to make the line pixels different with other part. So that we need a combined thesholded binary image by implementing color tranform and gradient. The gradient will introduce noise as it detects the color change rate, but it will benefit if the color tranform can't work in certain situation, so I sugest to use it as a supplementary method. Here is one example of my thresholded image:
![alt text][image3]

Step4
---
Use sliding windows to detect lines pixels and perform polyfit

In order to find the left line and right line, I need to get the histogram graph of the bottom half image. the peak of the histogram should be the start point of left line and right line. The I use a sliding window to search the left line and right line pixels by shifting the center of the window. We make a polyfit based on the found pixels for left line and right line. For the next frame, as we already know the fittef line for the last frame, we can use it as reference but will add one margin vallue ro re-search the pixels and polyfit to slightly adjust it to avoid blindly search every time. Sanity check will be preformed after each search and will reject the values if the cuvature changed to much or average x values changed a lot or the lines are not parallel.
The line for this implementation is in `line.ipynb` in function 'getfit(). Below is the example for identified lane-line pixels and fitted line:

![alt text][image5]

Step 5 Calculate the curvature of the line
---
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

Step 6
---
Map the detected lane boundaries into the original image. Here is the result:
![alt text][image6]


Step 7
---
Output the vehicle position and lane curvature. I use below code to output the curvature and vehicle position to the image:

```python
print(left_curverad, 'm', right_curverad, 'm')    

#calculate the current position in meters based on the polynominal line 's start point
nowposition=((left_fitx[-1]+right_fitx[-1])/2-640)*3.7/700
print("now position is: ",nowposition)

font = cv2.FONT_HERSHEY_SIMPLEX
bottomLeftCornerOfText = (230,60)
fontScale = 1
fontColor = (255,255,255)
lineType = 2

text=str(left_curverad)+  'm' +'   '+ str(right_curverad) +'m' +' pos '+ str(nowposition)

cv2.putText(result,text, bottomLeftCornerOfText, font, fontScale,fontColor,lineType)
cv2.imwrite('videof1/'+ image_name.split('/')[-1],cv2.cvtColor(result, cv2.COLOR_RGB2BGR))
```
Summary
---

This project is quite challenge as it requires a lot of time to debug and adjust the parameters and get it worked. For the project video, it's easy by adjusting a bit for different combination of color tranform and gradient. The challenge video is diffcult and in some frames even can't detect any lines. I performed histogram equalization twice to make it work. Here is my result for project video and chanllenge video:

[link to my  project video result](./project_video_submit.mp4)
[link to my  challenge video result](./challenge_video_submit.mp4)


