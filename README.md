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

### Install the FastCV SDK barcode_decoding.cpp
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
     3. Keep the given application folder ‘fastcvSimpleTest’ in <Hexagon_SDK_ROOT>/examples/common/
     ```
        $ cp fastcv-1-7-1_LinuxEmbedded/samples/fastcvSimpleTest/ <Hexagon_SDK_ROOT>/examples/common/
     ```
     4. Copy 32-bit libfastcv.a from libs provided in the fastcv sdk (fastcv-1-7-1_LinuxEmbedded\lib\32-bit\hard\libfastcv.a) to <Hexagon_SDK_ROOT>/examples/common/fastcvSimpleTest/lib/ folder.
     ```
        cp  fastcv-1-7-1_LinuxEmbedded\lib\32-bit\hard\libfastcv.a <Hexagon_SDK_ROOT>/examples/common/fastcvSimpleTest/lib/
     ```
     5. Open a new terminal from the root directory <Hexagon_SDK_ROOT> of the  Hexagon SDK and run below command        
      ```
          $ cd < Hexagon SDK root directory>
          $ source setup_sdk_env.source
          $ cd examples/common/fastcvSimpleTest
          $ make tree V=UbuntuARM_Release
     ```
     6. A folder with the name "UbuntuARM_Release" should get generated in the fastcvSimpleTest folder. It will contain the application’s binary file ‘fastcvSimpleTest’.

        
## To run application on the c610 board:

- Download a source repository which contains appliction code
     ```
        $ github clone <source repository>
        $ cd <source repository>
        $ cp fastcvSimpleTest <Hexagon_SDK_ROOT>/examples/common/
     ```    
     
- source the setup_sdk_env.source 
    ```
      $ cd `< Hexagon SDK root directory>
      $ source setup_sdk_env.source
      $ cd examples/common/fastcvSimpleTest
      $ make tree V=UbuntuARM_Release  
   ```
 - Note: while integrating opencv code into fastcv, we may get following errors (Similar error)
  ```
    /home/admin/Hexagon_SDK/3.5.2/tools/linaro/arm-linux-gnueabihf/include/c++/7.5.0/cstdlib:75:15: fatal error: stdlib.h: No such file or directory
    #include_next <stdlib.h>
               ^~~~~~~~~~
     To Fix this:Replace **#include_next** with **#include** in the cstdlib file from the path above.
```
   
- Remount the root directory writable:
  ```
           $ adb root
           $ adb remount
           $ adb shell mount -o remount,rw /
   ```
-  Push the fastcvSimpleTest binary file and input image to the target:
   ```
          $ adb push UbuntuARM_Release\ship\fastcvSimpleTest  /data/barcode
          $ adb push sampleImage  /data/barcode/
   ```
- Change bin permissions and execute the application executable:
   ```
          $ adb shell
          /# chmod +x /data/barcode/fastcvsampleTest
          /#  cd /data/barcode/
          to test with sample image  
          /# ./fastcvSimpleTest  sampleImage/barcode_img.jpg
          to test with qcs610 camera 
          /# ./fastcvSimpleTest camera
   ```
Then barcode numbers are displayed on the terminal.      
