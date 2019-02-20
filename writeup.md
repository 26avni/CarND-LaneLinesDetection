# **Finding Lane Lines on the Road** 

---
### Objective

The goal of this project is to develop a pipeline to be able to identify and highlight the lane lines in a given video.

The project as a whole performs the below tasks - 
* Takes a video as an input
* Identifies lane lines using helper functions and code
* Filters/Averages/Extrapolates the detected line segments
* Displays the lane lines marked in an output video

[//]: # (Image References)
[image_original]: ./test_images/whiteCarLaneSwitch.jpg "Original"
[image_gray]: ./test_images_output/whiteCarLaneSwitch_gray.jpg "Grayscale"
[image_gaussian]: ./test_images_output/whiteCarLaneSwitch_gaussian_blur.jpg "Gaussian"
[image_canny]: ./test_images_output/whiteCarLaneSwitch_canny_edge_detection.jpg "Canny"
[image_region]: ./test_images_output/whiteCarLaneSwitch_desired_area.jpg "Region of Interest"
[image_Hough]: ./test_images_output/whiteCarLaneSwitch_hough_transform.jpg "Hough Transformation"
[image_Final]: ./test_images_output/whiteCarLaneSwitch_final_output.jpg "Final Image"

---
### Lane Finding Pipeline 

My Pipeline consists of 7 steps - 

1. Converting the image to grayscale. 
``` gray_image = grayscale(image) ```

![alt_text] [image_gray]

2. Applying Gaussian Blur to the grayscaled image.
``` 
kernel_size = 5
blur_gray_image = gaussian_blur(gray_image,kernel_size) 
```

![alt_text] [image_gaussian]

   I chose the kernel value as 5

3. Applying Canny Edge Algorithm.
```
low_threshold = 50
high_threshold = 150
edges = canny(blur_gray_image,low_threshold,high_threshold)
```

![alt_text] [image_canny]

   I chose low_threshold value as 50 and high_threshold value as 150

4. Selecting region of Interest.
   Since, more than half of the image is covered by sky, we can select a quadrilateral region where we'll be looking for lane lines
```
imshape = edges.shape
vertices = np.array([[(100,imshape[0]),(450,320),(500,320),(imshape[1],imshape[0])]], dtype=np.int32)
masked_image = region_of_interest(edges,vertices)
```

![alt_text] [image_region]

   As per analysis, I chose the following coordinates - 
       (100,imshape[0]),(450,320),(500,320),(imshape[1],imshape[0])

5. Applying Hough Transformation.
```
rho = 2
theta = np.pi/180 
threshold = 30
min_line_len = 70 
max_line_gap = 40
lines = hough_lines(masked_image, rho, theta, threshold, min_line_len, max_line_gap)
```
   I chose the following values for the parameters - 
       rho = 2
       theta = np.pi/180 
       threshold = 30
       min_line_len = 70 
       max_line_gap = 40
       
6. Implementing draw_lines to filter/average/extrapolate the detected lines.   

![alt_text] [image_hough]

7. Finally, displaying the result using weighted_img
``` final_image = weighted_img(lines,image) ```

![alt_text] [image_final]

---
### draw_lines(img, lines, color=[255, 0, 0], thickness=10)

The objective of this method is to optimize the detected lines by averaging/extrapolating in order to make them more clear.

I have followed the below approach - 

1. For each line in lines, calculate slope and intercept.
$ y = mx + c $

2. Check if the slope for the particular line is negative or positive. Accordingly, store x coordinate, y coordinate, slope and intercept in their respective array.

3. Calculate the mean slope and intercept for both left lane lines and right lane lines.

4. Calculate the minimum and maximum values for x and y coordinates for both left lane lines and right lane lines.

5. For the hightest point in desired region, y coordinate will be having minimum value for both left and right lane lines.
    We can find the minimum value of y coordinate for both right and left lines from the array.
   Using these values of y along with mean slope and mean intercept, the x coordinates for highest point can be calulated for both left and right lane lines.
$ x = int((y-c)/m) $

6. We're supposed to connect the lane lines till the bottom of the image. The y coordinate will be equal to the width of the image.
``` 
ishape = img.shape 
```
   ``` ishape[0] ``` will give the highest possible value for y coordinate.
   We can use this value for y coordinate of bottom of the image and calcuate the x coordinate using mean slope and intercept values.
  
7. Now, since we've succeeded in finding both the end points for both lane lines, we can draw the lines
```
cv2.line(img, (left_x1_down, ishape[0]), (left_x2_up, min_left_y), color, thickness)
cv2.line(img, (right_x1_down, ishape[0]), (right_x2_up, min_right_y), color, thickness)
```

---
### Potential Shortcomings

1. It is not an optimal solution for detecting curved lines since it takes mean and draws a straight line.

---
### Possible Improvements

1. A better model with higher accuracy could be developed which takes into account the data from earlier detections and their accuracy in order to adjust parameters accordingly.
