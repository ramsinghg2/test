# Develop an Smart Camera use case: Barcode Reader application on C610

## About the project
- In this project, we are performing barcode region detection and recognition of the EAN-13 type barcode in the given image. 
- Application source tree on the host system:
           fastcvSampleTest/
                                 inc/
                                        - fastcv.h
                                         stb/
                                               -  stb_image.h
                                               -  stb_image_write.h 
                                  lib/                                  
                                         - contain fastcv and opencv libraries
                                  src/
                                         - fastcvSampleTest.cpp
                                  sampleImage/
                               Contain barcode images for testing                       
                                  Makefile
                                  UbuntuARM.min
                                                                                      
## Dependencies
- Ubuntu System 18.04 or above
- Install Adb tool (Android debugging bridge) on host system

## Prerequisites
- Setting up the Fastcv sdk on the host system as given in the Qualcomm document.

### Install the FastCV SDK 
1)  Download FastCV SDK from url https://developer.qualcomm.com/software/fast-cv-sdk/tools and select particular version named v1.7.1 for Linux Embedded
2)  For FastCV installation and compilation, follow the below steps
  ```
      $ chmod 777 fastcv-installer-linuxembedded-1-7-1.bin
      $ ./fastcv-installer-linuxembedded-1-7-1.bin     
  ```
 3) To build application in fastcv:
    1. Download Hexagon DSP SDK from Qualcomm developer network using url https://developer.qualcomm.com/software/hexagon-dsp-sdk
    -To install Hexagon sdk follow below steps
    ```
       $ chmod 777 qualcomm_hexagon_sdk_3_5_2_eval.bin
       $ ./qualcomm_hexagon_sdk_3_5_2_eval.bin
    ```
    
     2. Download  gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar from https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/
     - Extract the tar file 
     ```
        $ tar xvf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar
     ```
     - copy folder gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf to <Hexagon_SDK_ROOT>/tools/ folder.  
     ```
        $ cp -r gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/ <Hexagon_SDK_ROOT>/tools/
     ```
     
     - Rename gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf folder to linaro
      ```
         $ mv gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf linaro
      ```
     3. Keep the application  folder ‘fastcvSimpleTest’  in  <Hexagon_SDK_ROOT>/examples/common/. 
       Copy 32-bit  libfastcv.a from libs provided in the fastcv sdk (fastcv-1-7-1_LinuxEmbedded\lib\32-bit\hard\libfastcv.a) to <Hexagon_SDK_ROOT>/examples/common/fastcvSimpleTest/lib/ 
               Folder.

           4. Open a new terminal from the root directory    
                <Hexagon_SDK_ROOT> of the  Hexagon SDK and run setup_sdk_env.source. This script configures the local environment. These changes are not persistent in the terminal instance, so you must run  setup_sdk_env.source on each terminal you want to develop in.   
             
                   $ cd < Hexagon SDK root directory>
               $ source setup_sdk_env.source

            5. Change the directory to fastcvSimpleTest
                $ cd examples/common/fastcvSimpleTest
                Execute command below command
                $ make tree V=UbuntuARM_Release

          6. A folder with the name "UbuntuARM_Release" should get generated in the fastcvSimpleTest folder. It will contain the application’s binary file ‘fastcvSimpleTest’.


Introduction to Barcode: 

     Barcode is an image having a shape like a square or rectangle which consists of a sequence of parallel black lines and white spaces of various widths.  Barcodes are placed on products for the quick identification. 1D barcodes are used to store text information, like product size, type and color. 
 
In this project we are decoding EAN barcodes of type 1-D.
EAN is one of the standardized barcodes which is marked on most commercial products which are currently available at the stores. EAN barcodes only represent the digits 0–9, whereas some other barcodes  can represent additional characters along with digits
Following figure represents EAN-13 data composition. 



(1) Country code
It tells about the country 's name.

(2) Manufacturer code
It tells the original seller's name.
Manufacturer code is applied for registration at the code center of each country in order to obtain it. EAN code can be used only after the manufacturer code is obtained.
(3) Product item code
It identifies the product. The different products of the same manufacturer have different product item codes.
(4) Check digit
    It is used to verify whether a  barcode has been scanned correctly or not.
EAN Decoding involves 2 steps:
Barcode recognition
Barcode decode
 Barcode recognition:
        Input image is converted to a grayscale image. On the resulting image apply morphological operations and then apply contours to get the region of interest of barcode.
 Barcode decode:
Digits in the barcode are split into three groups such that : First digit, the first group of 6, the last group of 6.
Leaving the first digit, each digit of barcode is represented by 4 bars, such that white and black are alternate for the first group of 6 , black and white are alternate for the last group of 6.
Each group of 4 bars has the bar width as 7, whereas each bar in the group of 4 can be any value from 1 to 4. Binary values of this 7 bit value are compared with ‘L’ ‘G’ ‘R’ Table and barcode numbers are extracted. 
Steps to run the application: 
         
  To run application on the c610 board:

Remount the root directory writable:
$ adb root
$ adb remount
$ adb shell mount -o remount,rw /

Push the fastcvSimpleTest binary file and input image to the target:
          $ cd fastcvSimpleTest/         
          $ adb push UbuntuARM_Release\ship\fastcvSimpleTest    /data/barcode

         $ adb push sampleImage  /data/barcode/

Change bin permissions and execute the application executable:
           $ adb shell
          /# chmod +x /data/barcode/fastcvsampleTest
            /#  cd /data/barcode/
           /# ./fastcvSimpleTest  sampleImage/barcode_img.jpg

Then barcode numbers are displayed on the terminal.      
