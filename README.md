**Calibration-of-a-C-arm-X-ray-device-Winter-school**
  Authors: Shuai Zhang

**1. Datasets**

  The original images captured from the C-arm device can be found in the folder "original_images_uncropped", the size is 1280 x 1024;
  The cropped images can be found in the folder "uncalibrated", the size is 1024 x 1024.
  You should use the cropped images since the original images contain extra border illustrating some patient related information.
  
  The calibration phantom consists of an acrylic plate (10cm x 10 cm, thickness 4mm) and 25 stainless steel spheres (diameter: 3mm).
![image](https://user-images.githubusercontent.com/25130068/123815919-741ee380-d929-11eb-8648-9dbe8c5b00c3.png)

**2. Prerequisites**

  We have tested the code in Ubuntu 18.04, but it should be easy to compile in other platforms.
   We use OpenCV to manipulate images and features. OpenCV dowload and install instructions can be found at: https://docs.opencv.org/3.2.0/d7/d9f/tutorial_linux_install.html Required 3.2.0.
   
   
 **3. Building DefSLAM library and examples**

  Clone the repository:

  git clone https://github.com/zsustc/Calibration-of-a-C-arm-x-ray-device-Winter-school.git
  Execute:

  mkdir build
  
  cd build
  
  cmake .. 
  
  make

  This will create the executables x_ray_calibration and x_ray_calibration in the current folder.

**4. Settings**

Read the "help()" function in the provided code to learn about a camera calibration sample and also the example of command line for calibration from a list of stored images. 
"This is a camera calibration sample.\n"

        "Usage: calibration\n"
        "     -w=<board_width>         # the number of inner corners per one of board dimension\n"
        "     -h=<board_height>        # the number of inner corners per another board dimension\n"
        "     [-pt=<pattern>]          # the type of pattern: chessboard or circles' grid\n"
        "     [-n=<number_of_frames>]  # the number of frames to use for calibration\n"
        "                              # (if not specified, it will be set to the number\n"
        "                              #  of board views actually available)\n"
        "                              # (used only for video capturing)\n"
        "     [-s=<squareSize>]       # square size in some user-defined units (1 by default)\n"
        "     [-o=<out_camera_params>] # the output filename for intrinsic [and extrinsic] parameters\n"
        "     [-op]                    # write detected feature points\n"
        "     [-oe]                    # write extrinsic parameters\n"
        "     [-zt]                    # assume zero tangential distortion\n"
        "     [-a=<aspectRatio>]      # fix aspect ratio (fx/fy)\n"
        "     [-p]                     # fix the principal point at the center\n"
        "     [-v]                     # flip the captured images around the horizontal axis\n"
        "                              # [input_data] string for the video file name\n"
        "     [-su]                    # show undistorted images after calibration\n"
        "     [input_data]             # input data, one of the following:\n"
        "                              #  - text file with a list of the images of the board\n"
        "                              #    the text file can be generated with imagelist_creator\n"
        "                              #  - name of video file with a video of the board\n"
        "                              # if input_data not specified, a live view from the camera is used\n"

Using the "imagelist\_creator" function to create the xml or yaml list of stored images. You can also create the xml list of images manually using text editor.

" \nexample command line for calibration from a list of stored images:\n"

"   imagelist_creator image_list.xml *.png\n"

"   calibration -w=4 -h=5 -s=0.025 -o=camera.yml -op -oe image_list.xml\n"

" where image_list.xml is the standard OpenCV XML/YAML\n"

" use imagelist_creator to create the xml or yaml list\n"

" file consisting of the list of strings, e.g.:\n"

" \n"
"<?xml version=\"1.0\"?>\n"
"<opencv_storage>\n"
"<images>\n"
"view000.png\n"
"view001.png\n"
"<!-- view002.png -->\n"
"view003.png\n"
"view010.png\n"
"one_extra_view.jpg\n"
"</images>\n"
"</opencv_storage>\n";

**5. Define real world coordinates with phantom pattern**

**In the process of calibration we calculate the camera parameters by a set of know 3D points (X_w, Y_w, Z_w) and their corresponding pixel location (u,v) in the image.**
The pattern feature world coordinates are fixed by the phantom (checkerboard) pattern that is attached to a platform. Our 3D points are the centers in the grid of circles in the phantom. Any center of cricles on the above board can be chosen to the origin of the world coordinate system. The X_W and Y_W axes are along the platform, and the Z_W axis is perpendicular to the platform. All points on the phantom are therefore on the XY plane, which means Z_W=0.

For the 3D points we photograph the phantom pattern with known dimensions at many different orientations. The world coordinate is attached to the phantom and since all the center points lie on a plane, we can arbitrarily choose Z_w for every point to be 0. Since points are equally spaced in the phantom, the (X_w, Y_w) coordinates of each 3D point are easily defined by taking one point as reference (0, 0) and defining remaining with respect to that reference point.

In the foler "uncalibrated", we have multiple images of the phantom by moving the camera and keep the phantom static.

**6.Find 2D coordinates of phantom**

Since we got multiple of images of the phantom and we also have calculated the 3D location of points on the phantom in world coordinates in the last step. The next thing we need to do is to calculate the 2D pixel locations of these centers in the grid of circles in the images. OpenCV provides a builtin function called \textbf{cv::findCirclesGrid} that looks for a phantom and returns the coordinates of the detected centers. 
The function attempts to determine whether the input image contains a grid of circles. If it is, the function locates centers of the circles. The function returns a non-zero value if all of the centers have been found and they have been placed in a certain order (row by row, left to right in every row). Otherwise, if the function fails to find all the corners or reorder them, it returns 0.
Sample usage of detecting and drawing the centers of circles: :
@code
    Size patternsize(7,7); //number of centers
    Mat gray = ....; //source image
    vector<Point2f> centers; //this will be filled by the detected centers

    bool patternfound = findCirclesGrid(gray, patternsize, centers);
  
  **CV_EXPORTS_W bool findCirclesGrid( InputArray image, Size patternSize,
                                   OutputArray centers, int flags = CALIB_CB_SYMMETRIC_GRID,
                                   const Ptr<FeatureDetector> &blobDetector = SimpleBlobDetector::create());**

**7.Calibrate Camera**
  
  The final step of calibration is to pass the 3D points in world coordinates and their 2D locations in all images to OpenCV’s calibrateCamera method.
  
  **CV_EXPORTS_W double calibrateCamera( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints, Size imageSize,
                                     InputOutputArray cameraMatrix, InputOutputArray distCoeffs,
                                     OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                     int flags = 0, TermCriteria criteria = TermCriteria(
                                        TermCriteria::COUNT + TermCriteria::EPS, 30, DBL_EPSILON) );**
  
where,

objectPoints  A vector of vector of 3D points.
imagePoints	 A vector of vectors of the 2D image points.
imageSize 	Size of the image
cameraMatrix	Intrinsic camera matrix
distCoeffs	Lens distortion coefficients.
rvecs	Rotation specified as a 3×1 vector. The direction of the vector specifies the axis of rotation and the magnitude of the vector specifies the angle of rotation.
tvecs	3×1 Translation vector.
  
 **8. Optional**
  
  Trying different configuring parameters to compare the different returned re-projection error and validate the calibrated parameters.
