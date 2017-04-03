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

[imageDistort]: ./camera_cal/calibration5.jpg "Distorted"
[imageUndistort]: ./output_images/undistorted.jpg "Undistorted"
[imageBinarized]: ./output_images/binarized.jpg "Binarized"
[imagePerspective]: ./test_images/test2.jpg "Perspective Image"
[imageTransformed]: ./output_images/birdview.jpg "Bird View"
[imageQuadratic]: ./output_images/quadraticfit.jpg "Quadratic Fit"
[imageOutput]: ./output_images/output.jpg "Output"
[video1]: ./out.mp4 "Video" 

---
Please find all the code mentioned beyond in the Jupyter Notebook "AdvancedLaneFinding.ipynb".

### 1. Camera Calibration

In the first cell of the Notebook the camera calibration coefficients get calculated. I use some images of a chessboard pattern acquired in different angles for this. The corners of the squares can be automatically found by the OpenCV method 'findChessboardCorners' and save all the twodimensional corner coordinates in the array `imgpoints`. The array `objpoints` holds the undistorted threedimensional coordinates of the chessboard pattern in the world. They start with (0,0,0) in the upper left and end with (9,6,0) in the lower right, asuming they all lay in the same z plane. Finally the method `calibrateCamera` takes object and image points and calculates the distortion coefficients. The calibration images 1, 4 and 5 can't be used for calibration since not all corners are visible, but the result with the remaining images is sufficient. By using the method `undistort` to an distorted image one can get the following result: 

![Distorted Image][imageDistort]
![Undistorted Image][imageUndistort]

### 2. Binarize Image

In order to find lane lines, every unnecessary information has to be filtered out (well, at least as much as possible). For this purpose cell 3 holds some usuful methods for experimenting with gradient and color space filtering. The final binarizing pipeline I came up with can be found in cell 4 in the method 'binarize_image'. It uses the following techniques:

| Nr. | Technique | Input | Kernel Size / Channel | Low/High Threshold |
|:---:|---|:---:|:---:|:---:| 
| 1. | Color Space Threshold | Input Image | V Channel (HSV) | 210, 255 |
| 2. | Direction of Gradient Conversion | 1. | 9 | 0.7, 1.3 |
| 3. | Gray Conversion | Input Image | - | - |
| 4. | Gradient Conversion (x) | 3. | 9 | 15, 255 |
| 5. | Color Space Threshold | Input Image | B Channel (RGB) | 180, 255 |

The Result is a combination of the output of Numbers 2, 4 and 5.

![Binarized Image][imageBinarized]

### 3. Transform Image

Cell 5 contains the code to calculate the transformation matrix to make a perspective transformation on the input image to get bird view of the lane. `vert_in` holds four points of the right and left lane line which will be transformed to the points in `vert_out`. After this the lane lines should be parallel as can be seen in the following example.

![Original Image][imagePerspective]
![Bird View][imageTransformed]

### 4. Fit Polynomial

Once the binarized image is in bird view, the pixels identifying a lane line have to be detected. First I'm looking for the start of the lane line at the bottom of the image. I split the image in a left and a right sector containing only the lower fourth of the image. To not detect the edge of the road, the left fifth of the left sector and the right fifth of the right sector will be discarded. Next, I sum up the remaining columns and convolve them with window of ones. This results in a smoothed column histogram. The maximum marks the beginning of the lane line. From this point the same technique will be used layer by layer moving upwards the image, resulting in a position of the lane lines centroid for each layer. The method `filter_lane_pixels` takes an image and the centroids position to seek all pixels within a window around each centroid. Those are the pixels considered to be lane line pixels.

The next step is to perform a quadratic fit on those points to get a second grade polynomial, like it's done in the method `quadratic_fit`. Here is a example with the outcome.

![Quadratic Fit][imageQuadratic]

### 5. Final Pipeline

I combine all the steps above in cell 8 in the method `process`. The following steps will be deployed on every image:

1. Udistort image
2. Binarize and undistort image
3. Transform to bird view
4. Find centroids for each lane line for each layer
5. Find lane line pixels along the centroids
6. Perform quadratic fit on lane line pixels
7. Calculate the fitted lane line (one pixel per line for each y coordinate)
8. Calculate the radius of each lane line
9. Check validity
10. Color the space between the lane lines
11. Transform the colored space back on the original
12. Calculate the eccentricity of the car of the lanes center 

#### Validity check

To check if the current fit is valid, I check if the width between the found lane lines is within a lower and upper threshold. 2.5 and 3.5 m appeared to be values that work just fine.

#### Calculating the radius and the eccentricity
To get the radius the quadratic coefficients have to be calculated again in with metrical units passed to the method `get_radius`.

The position of the car is considered to be in the center of the image. The deviation of the center between the detected lane lines and the image center gives the eccentricity of the car.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion



