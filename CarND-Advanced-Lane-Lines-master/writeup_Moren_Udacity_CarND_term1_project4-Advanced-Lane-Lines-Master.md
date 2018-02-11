## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./test_images/test3.jpg "Original"
[image2]: ./test_images/test3.jpgundistorted_road_image.jpg "Undistorted"
[image3]: ./test_images/test3.jpgpipelined_road_binary.jpg"Road "Binary Example"
[image4]: ./test_images/test3.jpgwarped_road_image_birdview.jpg "Birdview Example"
[image5]: ./test_images/test3.jpgresulting_polyfit_template_created.jpg "Fit Visual"
[image6]: ./test_images/test3.jpgundistorted_road_image_lined_commented.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the function corners_unwarp the IPython notebook located in "./Udacity_CarND_Project_4-for-submission.ipynb".  

The function corners_unwarp(images, nx, ny) returns warped_input, M, newcameramtx, dist, mtx

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Each image that will be fed into the pipeline must first be corrected for distortion. This is to ensure that the image processing that follows will give reliable results.  Distortion can add noise to the computation of the radius of curvature, for example.  It can also influence the efficiency of the lane search filter using past knowledge.

The function correct_dist(fname, newcameramtx, dist, mtx) returns the original image and the original undistorted image
The function correct_dist_video has the same purpose, only is tailored for the video input

First let's have a look at the original image:
![alt text][image1]

Then here's the same image corrected for camera distortion:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The function pipeline(img2, s_thresh, sx_thresh) computes and returns a color mask "color_binary"

The binary thresholding is based on the scaled soebel calculations.  Here's an example of my output for this step.  (note: this is from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is performed by a function called 'bird_view_transform'.
The function bird_view_transform(undist_road_view, corners) returns warped_to_birdview, M, Minv

This function expects an image to transform, along with the source corners identified as the referenced corners to use in the transformation.  I have named the input image to remind the user that the input should be corrected for distortions before being transformed.
The output are respectively the transformed image from a birds perspective, and the matrices for performing either the transformation or the inverse transformation. the inverse matrix is a very useful (automatic) input for a similar function: road_view_transform.

I chose to hardcode the source points in the main code execution (cell 16, which begins with the comment: #test the functions proivded above) in the following manner:

these are the corners identified in the picture to perform the bird view transform (width, height), 
starting with the top right corner, top left corner, bottom left corner, bottom right corner

corners = np.float32(
    [[585,450],
     [715,450],
     [1210,600],
     [360,600]])

and in the bird_view_transform function, I have defined the default dst corners located in the resulting transformed image:
offset = 100

dst = np.float32(
    [[offset, offset],
    [img_size[0]-offset,offset],
    [img_size[0]-offset,img_size[1]-offset],
    [offset, img_size[1] - offset]])

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
The image fed into the bird_view_transform was previously thresholded.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

I have a 'do-all' function called template_creator
The function template_creator(window_centroids, warped_image, window_width, window_height, margin) 
returns: template, warpage, resulting_polyfit_template, text_to_print

It calls on the functions:
* window_mask, the function which creates the definition for a window mask for researching the window centroids
* find_window_centroids, which will apply the window mask to identify the image areas where the left and right lanes were detected.
* poly_image_extractor, which using the birdview tranformed image along with the knowledge of the window centroids location fits a polynom to both left and right lanes detected.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this at the end of the function poly_image_extractor (cell 14).  The function poly_image_extractor also prepares the text that will be annexed to the image for curvature.

here are the results I have obtained when computing both curvature and car position

    straight_lines1.jpg ####
    evaluated curvature left: 7029.93413351m 
    evaluated curvature right: 4043.96929774mlane width in meters : 3.61
    position of the car with respect to the lane center in meters : 1.7

    straight_lines2.jpg ####
    evaluated curvature left: 6472.86968423m 
    evaluated curvature right: 19721.3644247mlane width in meters : 3.71
    position of the car with respect to the lane center in meters : 1.65

    test1.jpg ####
    evaluated curvature left: 1154.40251838m 
    evaluated curvature right: 516.536506541mlane width in meters : 3.89
    position of the car with respect to the lane center in meters : 1.56

    test2.jpg ####
    evaluated curvature left: 8565.77377887m 
    evaluated curvature right: 575.3200006mlane width in meters : 3.43
    position of the car with respect to the lane center in meters : 1.79
    
    test3.jpg ####
    evaluated curvature left: 685.576344119m 
    evaluated curvature right: 599.106871589mlane width in meters : 3.7
    position of the car with respect to the lane center in meters : 1.65

    test4.jpg ####
    evaluated curvature left: 643.099915273m 
    evaluated curvature right: 489.580036092mlane width in meters : 3.64
    position of the car with respect to the lane center in meters : 1.69

    test5.jpg ####
    evaluated curvature left: 553.191996135m 
    evaluated curvature right: 492.450970708mlane width in meters : 3.9
    position of the car with respect to the lane center in meters : 1.55

    test6.jpg ####
    evaluated curvature left: 1368.14615124m 
    evaluated curvature right: 580.090657912mlane width in meters : 4.03
    position of the car with respect to the lane center in meters : 1.49


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

The main technical problem I have faced in this project is due to not being a regular programmer, and requiring a lot of time to debug my script when an obscure error would be thrown.  
Additionally, I notice that the downside to using Jupyter notebooks for coding is that python does not execute as smoothly as with other environments.  
For example, I would regularly have to cast array types individually; the code would throw back an error if I would try to pack in such a command within another set of commands.

Another example is that on individual test images I have managed to integrate text providing the curvature of the road, yet the same processing did not yield a text overlay in my video output. 
A third example is that I did not manage to implement the filling of the lane using cv2.fillPoly due to the occurence of the error: error: (-215) cn <= 4 in function cv::scalarToRawData
 
Areas where I can certainly improve my line detection algorithm:
1) integrate a coherency check using the knowledge of the line width and additional algebra to check if left lane and right lane lines are close to parallel and properly spaced

2) integrate a check using past knowledge of the lanes' positions to stabilize the polynome fitted to each lane.

3) the checks from 1) and 2) can be further integrated to an algorithm that permits to artificially maintain both lane lines as long as at least one lane is reliably found.

4) Robustness in the accelerated search field as currently implemented is too sensitive to noise in the neighboring pixels.  
An idea for improvement would be to rather look in the vertical dimension instead of the horizontal dimension.  
Another idea for improvement would be to keep the original search area, but reduce its width further.

5) For some reason, likely linked to the planeity of the road (when going a bit more uphill or downhill likely), the perspective transform does not yield an easy image to search for lanes. 
The result are lanelines that are clearly not parallel, independantly of adjusting the perspective transform vertices. 
Further difficulties that are noteworthy in this image are due to the shaddow casted by the tree to the left of the road, followed by a change in quality of the surface of the road (newer, darker...)
