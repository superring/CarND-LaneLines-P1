# **Finding Lane Lines on the Road** 

## Writeup
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/solidWhiteCurve.png "solidWhiteCurve"
[image2]: ./writeup_images/solidWhiteRight.png "solidWhiteRight"
[image3]: ./writeup_images/solidYellowCurve.png "solidYellowCurve"
[image4]: ./writeup_images/solidYellowCurve2.png "solidYellowCurve2"
[image5]: ./writeup_images/solidYellowLeft.png "solidYellowLeft"
[image6]: ./writeup_images/whiteCarLaneSwitch.png "whiteCarLaneSwitch"
[image7]: ./writeup_images/option_challenge.png "option_challenge"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps below. 

First, I converted the images to grayscale using the helper function grayscale().

    image = mpimg.imread(filename)
    gray = grayscale(image)

Second, I used the helper function gaussian_blur() to apply the Gaussian Smoothing on the grayscale image.

    kernel_size = 3
    blur_gray = gaussian_blur(gray, kernel_size)

Third, I used the helper function canny() to detect and draw the edges of the image, while all other blacked out. My tuned parameters are:

    low_threshold = 50
    high_threshold = 150
    edges = canny(blur_gray, low_threshold, high_threshold)

Forth, I defined a four sided polygon to mask the area of interest that I'm going to apply the detection.

    imshape = image.shape
    vertices = np.array([[(bot_left_corner_X,imshape[0]),(top_left_corner_X, vertices_top), (top_right_corner_X, vertices_top), (bot_right_corner_X,imshape[0])]], dtype=np.int32)
    masked_edges = region_of_interest(edges, vertices)
I tuned the coordinates of the 4 corners of the polygon with the parameters below, which are defined in the 'Helper Functions' as global variables. Another reason I used the global variables is that I realized I might be able to adjust the polygon shape under the occassion of curving for the optional challenge.

    vertices_top = 330
    top_left_corner_X = 440
    top_right_corner_X = 550
    bot_left_corner_X = 10
    bot_right_corner_X = 890

Finally, I tuned the parameters of the Hough Transform.

    rho = 1 # distance resolution in pixels of the Hough grid
    theta = np.pi/180 # angular resolution in radians of the Hough grid
    threshold = 15     # minimum number of votes (intersections in Hough grid cell)
    min_line_length = 40 #minimum number of pixels making up a line
    max_line_gap = 20    # maximum gap in pixels between connectable line segments

Then I drawed the Hough Lines using help function hough_lines(), as well as weighted_img() to make the lines semi-transparent.
 
    # Draw Hough Lines
    line_image = hough_lines(masked_edges, rho, theta, threshold, min_line_length, max_line_gap)
    # combine the line image and the initial image with lines semi-transparent
    lines_initial = weighted_img(line_image, image)

Here're my results of the test images:

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by saperating the Hough Lines into 2 categories: left-side lines and right-side lines.

To do so, I calculated the slopes of the lines with the equation below:

    m = (y2 - y1)/(x2-x1)

And if the calculated m satisfies:

    m > 0.2 and m < 0.8

The line will be on the left lane.

If the calculated m satisfies:

    m < -0.2 and m > -0.8

The line will be on the right lane.

After the categorizing, I used the numpy.polyfit() to calculate the fitted slopes and intercepts for the lines on both side.

    fit_Left  = np.polyfit(leftX, leftY, 1)
    fit_Right = np.polyfit(rightX, rightY, 1)

And to extrapolate the lines to the top and the bottom of lanes, I used the following calculation:

        top_Y_left = vertices_top
        top_X_left = int( (top_Y_left - fit_Left[1]) / fit_Left[0] )
        bot_Y_left = img.shape[1]
        bot_X_left = int( (bot_Y_left - fit_Left[1]) / fit_Left[0] )

Both the images test and the video test show a good result.


### 2. Identify potential shortcomings with your current pipeline

Since the vertices of my polygon of interest are fixed, one potential shortcoming would be what would happen when the car is curving, ascending or descending, there might be noises since other objects than lanes will get into the area of interest. 

Another shortcoming could be if a car gets inside the area of interest, there will also be noises which affect the line drawing.


### 3. Suggest possible improvements to your pipeline

The possible improvement would be to make the vertices of the polygon to be self-adjustable. 

I tried re-calculating the top vertices everytime after determing the top points on the lanes, in the draw_lines() function_

    # determine the top point of left lane
    top_Y_left = vertices_top
    top_X_left = int( (top_Y_left - fit_Left[1]) / fit_Left[0] )
    # recalculate the top-left vertex
    top_left_corner_X = top_X_left - 20

    # determine the bottom point of left lane
    bot_Y_left = img.shape[1]
    bot_X_left = int( (bot_Y_left - fit_Left[1]) / fit_Left[0] )
    # recalculate the bottom-left vertex
    bot_left_corner_X = bot_X_left - 20

    # determine the top point of right lane
    top_Y_right = vertices_top
    top_X_right = int( (top_Y_right - fit_Right[1]) / fit_Right[0] )
    # recalculate the top-right vertex
    top_right_corner_X = top_X_right + 20

    # determine the bottom point of right lane
    bot_Y_right = img.shape[1]
    bot_X_right = int( (bot_Y_right - fit_Right[1]) / fit_Right[0] )
    # recalculate the bottom-right vertex
    bot_right_corner_X = bot_X_right + 20

However this method still seems not accurate as shown below. 

![alt text][image7]

I might need to find another way of improvement. I also want to hear if there's any suggestions.
    