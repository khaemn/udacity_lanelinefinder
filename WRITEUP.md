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
[image08]: ./illustrations/lane_lines.jpg "Lane lines"
[image09]: ./illustrations/whiteCarLaneSwitch.jpg "Pipeline result"

---

### Reflection

### 1. The pipeline and the modified draw_lines() function.

My pipeline consists of 3 major steps preprocessing, lane finding, and summing the lanes
image with an input one.

The main task of the preprocessing is to highlight lane markings on a road as much as possible
and define a region of interest (ROI) for further processing. As the color of lane line markings
does not matter in scope of this project, I have discarded a color information and used HSL color
space, mixing luminocity and saturation channels (50/50). 
This approach (as opposed to a plain
grayscale conversion) saves more of the valuable information on image:

![alt text][image03]

Luminosity + Saturation mix above, direct Grayscale below (notice improved contrast):

![alt text][image04]

The mixed sat/lum grayscale image is then blurred with a small radius to smooth any small-detail noise:

![alt text][image05]

Then processed with Canny filter to detect edges

![alt text][image06]

And then cropped by a trapezoidal ROI:

![alt text][image07]


Preprocessed image is then passed to a HoughLines transformation to detect any lines, that fits
to its criteria. 
HoughLines might return many lines, which are irrelevant and caused by shadows or color changes
on the road, not related to lane markings. To filter out such lines, each point pair is converted
to a line equation coefficients (according to canonic `y = kx + b` equation, where `k` is the 
slope, and `b` is an offset from X axis). Now, while the car's camera remains close to the lane
center and the lane is either straight or has a large radius, both left and right lane lines
have useful properties:
 * they must have close (but mirrored) slope values
 * the absolute slope value should lie within a particular range (neither vertical, nor 
horizontal lines can not be the one we are looking for.)
 
On the test videos the slope of the right lane line is about -45 degrees, but due to inverted Y axis 
on the images, the real slope value is positive for right and negative for left lane line. Now it is
possible to filter out any line, that does not fit into an allowed slope range, and then average
all slopes and offsets of all 'left' and 'right' lines, receiving 2 line equations respectively.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by
adding the filtering algorithm described above, and then used the resulting equations to draw 2
lines starting from the image bottom up to a predefined top limit (it was adjusted for the test
videos to achieve the best appearance).

![alt text][image09]

The filtering and averaging algorithm has turned out to be robust enough to work on all test images
and videos, using exactly the same parameters.


### 2. Identify potential shortcomings with your current pipeline


The main goal of this project is to deliver a working solution; I haven't not concentrated
on the performance and high maintainability, so there are some known drawbacks:

* The `get_average_left_and_right_lanes()` function uses `for` loops, while the same 
functionality might have been achieved by using native numpy array operations, which are faster;
though, having simple loops allows a reviewer to catch the idea much easier.

* The NaiveBayesianFilter class is used as a global variable, as it seemed to be the optimal way
for fast prototyping, but this approach also requires resetting the globally declared filter
before starting each next run of the pipeline, for example, before each new test image or test
video is loaded to the pipeline.

* The `get_average_left_and_right_lanes()` function uses average (mean) values of a line slope
and offset, while having a *median* ones might be better from the accuracy point of view, as
there are significant glitches of these values when processing a noisy frame. I did
not implement median statistics gathering to keep the code simple... and it works fine anyway.


### 3. Suggest possible improvements to your pipeline

If I did need to deliver the pipeline as a production solution, I would have done the next steps:

* refactor `get_average_left_and_right_lanes()` function to use *median* instead of *average* and
operate native numpy array ops instead of loops.
* restructure the whole pipeline to a class, which would aggregate a filter instance (NaiveBayes or
another one, depending on requirements); this class would have a method for processing (e.g.
annotating) an image, and might be used both for processing images and video frames.
