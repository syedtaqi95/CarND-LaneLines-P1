# **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

Find my project on [Github](https://github.com/syedtaqi95/CarND-LaneLines-P1).

[//]: # (Image References)
[original]: ./writeup_images/original.png "original"
[grayscale]: ./writeup_images/grayscale.png "grayscale"
[white_pixels]: ./writeup_images/white_pixels.png "white_pixels"
[yellow_pixels]: ./writeup_images/yellow_pixels.png "yellow_pixels"
[image_whiteyellow]: ./writeup_images/image_whiteyellow.png "image_whiteyellow"
[image_blurred]: ./writeup_images/image_blurred.png "image_blurred"
[canny_edges]: ./writeup_images/canny_edges.png "canny_edges"
[canny_edges_masked]: ./writeup_images/canny_edges_masked.png "canny_edges_masked"
[line_image]: ./writeup_images/line_image.png "line_image"
[output_image]: ./writeup_images/output_image.png "output_image"

---

## **Lane Detection Pipeline**

The pipeline detects white and/or yellow road markings and annotates the original image/video with red lines where it detects a lane. I shall use the _whiteCarLaneSwitch.jpg_ file to illustrate the different stages of the pipeline.

Let's start with a comparison of the original image versus the output:

![original]![output_image]

I used the tools and functions provided in the lectures along with other techniques that I researched online to develop my lane detection pipeline. My code mainly makes use of the *matplotlib, numpy* and *cv2* Python libraries.

I will now explain how my pipeline works. It consists of 9 stages as detailed below:

**1. Convert the original image to grayscale.**

This simply uses OpenCV's *cvtColor* function to convert the image from RGB to grayscale. This is done to make it easier to extract the white pixels from the image using one colour channel instead of three.

![grayscale]![original]

**2. Extract white pixels**

I created a mask to extract the white pixels, i.e. all pixels with a grayscale value of 197 and above. I used OpenCV's *inRange* function to do this.

![white_pixels]![original]

**3. Extract yellow pixels**

The RGB colour space is not great at detecting yellow colours in images, hence the first thing I did was convert the image to the HSV (Hue-Saturation-Value) colour space. This was done using OpenCV's *cvtColor* function, similar to the grayscale conversion.

Next, I used the *inRange* function again to extract the yellow pixels from the HSV image. The thresholds I used were:

```python
yellow_thresh_low = (20,100,100)
yellow_thresh_high = (30,255,255)
```
This extracted the yellow lane on the left fairly accurately.

![yellow_pixels]![original]

**4. Combine white and yellow pixels**

I simply combined the white and yellow masks using OpenCV's *bitwise_or* function and applied it to the image using the *bitwise_and* function to get the result below.

![image_whiteyellow]![original]

**5. Gaussian blur**

Gaussian blur is suggested prior to applying Canny edge detection as it helps smooth out the image and reduce noise. I used the provided *gaussian_blur* function for this.

![image_blurred]![original]

**6. Canny edge detection**

Canny edge detection is a powerful technique to extract lines from an image. I tested the low and high thresholds until it worked fairly well for the test images.

![canny_edges]![original]

**7. Apply a region of interest**

This stage filters out all the lines detected outside our region of interest, which is simply a polygon roughly covering an area in the lower half of the image. The result is an image which only contains the lane edges.

![canny_edges_masked]![original]

**8. Hough transform and drawing the Hough lines**

I used the provided *hough_lines* function to apply the Hough transform and extract the Hough lines.

I wrote the *draw_lines_extrapolated* function to replace the original *draw_lines* function. This was done to extrapolate the lines and make the output similar to that in *P1_example.mp4*. 

First, I computed the slope of each Hough line and sorted them into left and right lanes. Next, I used numpy's *polyfit* function to generate a 1-degree polynomial (i.e. a line) based on the coordinates of the Hough lines for each lane. I found that I could use numpy's *poly1d* function to generate the y-values of the polynomial (rather than manually computing "y = mx + b"). I then selected coordinates such that the lines would extend till the image boundary and drew these lines using OpenCV's *line* function.

![line_image]![original]

**9. Annotate the image/frame**

The final step was simply using the provided *weighted_img* function to annotate the original image with the output of the previous stage to give the final output.

![output_image]![original]

## **Limitations**

As I developed my lane detection pipeline I became aware of the following limitations:

* Parameter tuning was a manual process and took a lot of trial and error to get right for the provided images.
* Related to the above point, the parameters are static and will not work well in different lighting conditions, road surfaces, environments etc.
* The pipeline does not take any system performance requirements into account, i.e. it assumes unlimited processing resources and unlimited time to detect lanes for each frame. This is fine for learning purposes, however a real autonomous vehicle needs to account for each millisecond of latency, because in some cases it could be the difference between life and death.
* This pipeline assumes that the lane markings are in good condition. This may not be the case in areas with badly maintained roads.

## **Suggested improvements**

* The pipeline does not work well with curved roads. One way to improve curved lane detection is to use Hough circles, and use a higher order polynomial in the *polyfit* function.
* The output in the videos is a bit "jerky". Implementing a linear filter on the output would make this smoother and improve accuracy.