# Advanced Lane Tracking

## Udacity Writeup Question Pointers
- Camera Calibration -> Detail Discussion/Camera Calibration
- Pipeline(test images)
    - 1 -> Detail Discussion/Feature Extraction Pipeline/Undestortion process
    - 2 -> Detail Discussion/Feature Extraction Pipeline/Feature Extraction
    - 3 -> Detail Discussion/Perspective Transform
    - 4 -> Detail Discussion/Curve Fitting
    - 5 -> Detail Discussion/Sanity Check
    - 6 -> Detail Discussion/Output
- Pipeline(video) -> Introduction
- Discussion -> Challenge Discussion

 

## Introduction
The goal  of this project is to identify lanes from a front camera of a car and highlight the region that is safe for the car to drive.

Here's a [link](https://youtu.be/DRTsc3jyWg0) to my video result.

1. Perform camera calibration from a given set of chessboard images taken from the same camera used to record the short clip. The calibration result is stored in a pickle file for read/load for undistortion.
2. Every frame in the video is transformed through a desinated pipeline that extract different features from the original frame.
3. Extract lane information using perspective transform and perform sanity check on them. 
    1. If lanes are valid, they will be recorded and used as future reference for faster lane extraction. 
    2. If lanes are not valid, they will be marked as bad frame and use the previous average lane information as valid lanes instead.
![extraction pipeline](https://i.imgur.com/MTTQVeh.png)

## Detail Steps

### Camera Calibration

- `objp`: a 3D coordinate array indicating where the chessboard corners should be. In this project, the chessboard is a 2D plane so all the z coordinates are set to 0. 
- `objpoints`: an array of coordinates copy from successfully detected chessboard corners in test image.
- `imgpoints`: an array of 2D pixel position of each of the corner detected on the successfully detected image.
- `mtx`, `dist`: the calibration and distortion coefficients generated from `cv2.calibrateCamera()`, which uses opjpoints and imgpoints for distortion calibration.
- `dist_pickle`: a global dictionary that stores `mtx`and `dist`. It will be dump to a pickle file as cam_cal.p for future usage.

Result from undistort:
![](https://i.imgur.com/a9fgoDp.jpg)

### Feature Extraction Pipeline
![tranformation pipeline](https://i.imgur.com/YsGdD0R.png)

1. Undestortion process

    All the in put image are raw image from camera, so they must receive undistort before furhter processing.
![](https://i.imgur.com/ZAL82E2.jpg)

2. Material Preparation
- `gray`: a grayscale image for gradient filtering
- `hls`: a HLS colored image for saturation filtering
3. Feature Extraction
    There are 3 methods in the extraction process, each method will output a binary map indicating the activated pixels.
    1. gradient fitlering on x axis
    2. gradient filtering on direction(dy/dx)
    3. saturation fitlering 
    Afterwards, the 3 binary maps are stacked together to form 2 clear lanes on it.

![](https://i.imgur.com/TcIk4lP.jpg)

### Perspective Transform
To perform perspective transform, 2 arguments are requried:
1. `src`: a hard-coded pixel locations that forms a quadrilateral, which covered both lane from close to far.
2. `dst`: a hard-coded pixel locations that forms a rectangle, which will be the transformed area.

Note: The pixel locations are picked according to the given clip and may need to adjust for different video.

The output file is varified by checking manually whether the lanes are parallel and the trasform matrix `M` and reverse matrix `Minv` are collected for futhre usage.

![](https://i.imgur.com/7ttzOb0.jpg)

### Curve Fitting
The fitting process has 2 different approaches, one is to find the fitting curve form scratch and the other one is to find the fitting curve using previous information.
1. Curve fitting from scratch:
    Given that all lanes near car will be fairly vertical to camera, in `find_lane_pixels()`, I used a histogram to collect the activated pixels in the bottom one-nineth part and use the most stacked locations as the staring points in . I then create 9 windows with one-nineth of the whole picture in height and 100 pixel in width to capture all activated pixcel iteratively. If the amount of the pixcels exceeded 50, I recenter the window to the average location of the activated pixels in current window. Finally, bt collecting all the activated pixels with 9 windows, they can be used to fit to a quadratic equation.

![](https://i.imgur.com/wkw5ARH.jpg)

2. Curve fitting with previous information:
    Given that all lane are continual, in `fit_poly_with_prev()`, I use the previous stored coefficients to create the center points for each y value and use these conter points to create windows  similar to the first part. The collected pixels will be fitted to a new quadratic equation.

### Sanity Check
For every fitted lane, `sanity_check()` can tell if the lane are badly or well detected. For those who failed the check, it is abandoned and the previous lane status will be used to generate the current lane indication. If the check failed continually for 15 times, the lanes will be fetched from scratch again.

The sainity check contained 3 parts:
1. curvature test
2. distance test
3. parallel test

### Output
After all the process above, revserse perspective and output the stacked image

![](https://i.imgur.com/3nmHL8R.jpg)


## Challenge Discussion
1. Hard-coded tuning
    The `src` and `dst` in the perspective transform is hard-coded without detecting the feasibility to the input clip. For my program to work, one must tune these hyperparameters according to the input clip before running the whole program. To overcome this challenge, another program like format transformer might automate the tuning part and solve this issue.

2. Objects covering lanes
    Using purely computer vision tools is impossible to detect objects that might cover the lanes. In the current process, they will be treated as the lane itself and wrong features will be reported. This can be solve using object detection to filter out their presence.


