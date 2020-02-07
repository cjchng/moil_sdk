# MOIL SDK 

MOIL SDK is collection of functions support c++ developments for fisheye image applications, tested both on ubuntu 18.04 and Raspberry Pi( Raspbian Buster ), gcc/g++ and OpenCV are required in the development. 

![moil_basic_steps](https://user-images.githubusercontent.com/3524867/73999970-65850480-49a1-11ea-9e0b-6b88d1d49fb7.jpg)

![semisphere](https://user-images.githubusercontent.com/3524867/74001393-61a7b100-49a6-11ea-96a0-112dbdeb7b05.jpg)

## 1. Development environment

For RaspberryPi, please download Raspbian opetating system image :
https://www.raspberrypi.org/downloads/raspbian/
following the installing steps : 
https://www.raspberrypi.org/documentation/installation/installing-images/
or tutorial :
https://oranwind.org/-raspberry-pi-win32-disk-imager-shao-lu-sd-qia-jiao-xue/

 If you already have Opencv installed, the following steps can be skipped. Opencv version 3.2.0 is recommented, it's the default version on both Ubuntu 18.04 and Raspbian Buster. 

	sudo apt update
	sudo apt upgrade
	sudo apt install build-essential cmake pkg-config
	sudo apt install libjpeg-dev libpng-dev libtiff-dev
	sudo apt install software-properties-common
	sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
	sudo apt update
	sudo apt install libjasper1 libjasper-dev
	sudo apt update
	sudo apt install libgtk-3-dev
	sudo apt install libatlas-base-dev gfortran
	sudo apt install libopencv-dev python-opencv

## 2. Includes

    #include "moildev.h"

## 3. API Reference

#### 3.1 Config 

    C++ : bool Config(string cameraName, double cameraSensorWidth, double cameraSensorHeight,
        double iCx, double iCy, double i_ratio,
        double imageWidth, double imageHeight, double calibrationRatio,
        double para0, double para1, double para2, double para3, double para4, double para5
        );
        
    Purpose : 
    
> Each fisheye camera can be calibrated and derives a set of parameters by MOIL laboratory, before the successive functions can work correctly, configuration is necessary in the beginning of program. 
    
    Parameters :
    
    . caneraName - A string to describe this camera
    . cameraSensorWidth - Camera sensor width (cm)
    . cameraSensorHeight - Camera Sensor Height (cm)
    . iCx - image center X coordinate(pixel)
    . iCy - image center Y coordinate(pixel)
    . i_ratio : Sensor pixel aspect ratio.
    . imageWidth : Input image width
    . imageHeight : Input image height  
    . calibrationRatio : input image with/ calibration image width
    . para0 .. para5 : calibration parameters

    Example:   
    
    #include "moildev.h"
    Moildev *md = new Moildev();
    // Raspberry Pi 220 degree fisheye camera
    md->Config("car", 1.4, 1.4,
        1320.0, 1017.0, 1.048,
        2592, 1944, 4.05,
        0, 0, 0, 0, -47.96, 222.86
        );

#### 3.2 AnypointM     

    C++ : double AnyPointM(float *mapX, float *mapY, int w, int h, double alphaOffset, double betaOffset,
    double zoom, double magnification);

    Purpose :
    
> To generate a pair of X-Y Maps for the specified alpha, beta and zoom parameters, the result X-Y Maps can be used later to remap the original fisheye image to the target angle image.   

    Parameters : 

    . mapX : memory pointer of result X-Map   
    . mapY : memory pointer of result Y-Map
	. w : width of the Map (both mapX and mapY)
	. h : height of the Map (both mapX and mapY)
	. alphaOffset : alpha offset 
	. betaOffset : beta offset
	. zoom : decimal zoom factor, normally 1..12
	. manification : input image width / calibrationWidth, where calibrationWidth can get by calling getImageWidth(), manification is normally equal to 1.  

	Example :

	#include <opencv2/opencv.hpp>
    #include "moildev.h"
    Moildev *md = new Moildev();
	// md->Config	
	int m_ratio = 2592/ md->getImageWidth();
    Mat mapX = Mat(1944, 2592, CV_32F);
	Mat mapY = Mat(1944, 2592, CV_32F);
	Mat image_input = imread( "image.jpg", IMREAD_COLOR);
	Mat image_result;
	md->AnyPointM((float *)mapX.data, (float *)mapY.data, mapX.cols, mapX.rows, 0, 0, 4, m_ratio);     
	cv::remap(image_input, image_result, mapX, mapY, INTER_CUBIC, BORDER_CONSTANT, Scalar(0, 0, 0));

#### 3.3 fastAnyPointM

	C++ : double fastAnyPointM(float *mapX, float *mapY, int w, int h, double alphaOffset, double betaOffset,
	double zoom, double magnification);

    Purpose :
    
> To generate a pair of X-Y Maps for the specified alpha, beta and zoom parameters, the result X-Y Maps can be used later to remap the original fisheye image to the target angle image. This function use fast algorithm to achieve similar result. However, some sampling artifacts appear with larger alphaOffset value. This function save about 50% computing time on Raspberry Pi.     

    Parameters : 

	same as 3.2 AnyPointM function

	Examples :

	same as 3.2 AnyPointM function

#### 3.4. PanoramaM

	C++ : double PanoramaM(float *mapX, float *mapY, int w, int h, double magnification, double alpha_max);

    Purpose :
    
> To generate a pair of X-Y Maps for the specified alpha, beta and zoom parameters, the result X-Y Maps can be used later to generate a panorama image from the original fisheye image.   

    Parameters : 

    . mapX : memory pointer of result X-Map   
    . mapY : memory pointer of result Y-Map
	. w : width of the Map (both mapX and mapY)
	. h : height of the Map (both mapX and mapY)
	. manification : input image width / calibrationWidth, where calibrationWidth can get by calling getImageWidth(), manification is normally equal to 1.  
	. alpha_max : max of alpha. The recommended vaule is half of camera FOV. For example, use 90 for a 180 degree fisheye images and use 110 for a 220 degree fisheye images.

	Example :

	#include <opencv2/opencv.hpp>
    #include "moildev.h"
    Moildev *md = new Moildev();
	// md->Config	
	int m_ratio = 2592/ md->getImageWidth();
    Mat mapX = Mat(1944, 2592, CV_32F);
	Mat mapY = Mat(1944, 2592, CV_32F);
	Mat image_input = imread( "image.jpg", IMREAD_COLOR);
	Mat image_result;
	md->PanoramaM((float *)mapX.data, (float *)mapY.data, mapX.cols, mapX.rows, m_ratio, 110);     
	cv::remap(image_input, image_result, mapX, mapY, INTER_CUBIC, BORDER_CONSTANT, Scalar(0, 0, 0));

## 3. Build 

	1. Ubuntu 18.04 :

	g++ -o mainmoil main.cpp moildev.a `pkg-config --cflags opencv` `pkg-config --libs opencv` 


	2. Raspberry Pi

	g++ -o mainmoil main.cpp moildev_rpi.a `pkg-config --cflags opencv` `pkg-config --libs opencv` 

## 4. Example project

[https://github.com.cjchng/mainmoil_6view](https://github.com/cjchng/mainmoil_6view)










