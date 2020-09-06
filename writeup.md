# **Finding Lane Lines on the Road** 

### Volodymyr Havrylov, 2020

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image01]: ./illustrations/luminosity.jpg "Luminosity channel"
[image02]: ./illustrations/saturation.jpg "Saturation channel"
[image03]: ./illustrations/gray.jpg "Grayscale sum of lum+sat"
[image04]: ./illustrations/truegray.jpg "Grayscale directly converted"
[image05]: ./illustrations/blurred.jpg "Blurred"
[image06]: ./illustrations/edges.jpg "Canny edges"
[image07]: ./illustrations/masked.jpg "Cropped to ROI"


---

### Reflection

### 1. The pipeline and the modified draw_lines() function.

My pipeline consists of 3 major steps preprocessing, lane finding, and summing the lanes
image with an input one.

The main task of the preprocessing is to highlight lane markings on a road as much as possible
and define a region of interest (ROI) for further processing. As the color of lane line markings
does not matter in scope of this project, I have discarded a color information and used HSL color
space, mixing luminocity and saturation channels (50/50). This approach (as opposed to a plain
grayscale conversion) saves more of the valuable information on image. The mixed sat/lum grayscale
image is then blurred with a small radius to smooth any small-detail noise, processed with Canny
filter to detect edges, and then cropped by a trapezoidal ROI.

![alt text][image1]
![alt text][image2]
![alt text][image3]

Preprocessed image is then passed to a HoughLines transformation to detect any lines, that fits
to its criteria. 
In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
