# AWS IoT C++/Python SDK integration into the QCS610 and Data posting to AWS IOT console

## About the project
 In this project, we are integrating the AWS IOT C++/Python sdk into the c610 platform and also create  a small application to notify the current status of camera to aws iot core console. The source tree of the project start with application directory (/home/user/aws_iot)

          aws-iot/ 
                       -  video_record.c
                       -  video_record.h
                       -  basic_pub_sub/ 
                                        - cJSON.c ,   cJSON.h , main.cpp, CMakeLists.txt
                       -  publish.py
                       -  certificates/
                                     - xxxx-certificate.pem.crt, xxxx-private.pem.key, root-CA.crt
                                             
## Dependencies

- Ubuntu System 18.04 or above
- Install Adb tool (Android debugging bridge) on host system
- Setup the QCS-610 Yocto build environment  
- Creation on amazon aws user account
- Install python 3.5 or above on the host system 

### **Note:** 
-  For python implementation, building on the yocto environment is not required. 

## Prerequisites

- Setting up the Application SDK on the host system.
- Camera Environment configuration setup on the target device.
- Setting up aws-iot device connect
- Compiling the aws-iot-c++ sdk on yocto environment

### 1. Setting up the Application SDK on the host system:

   - Download the Application SDK from below url:
      -  https://thundercomm.s3.ap-northeast-1.amazonaws.com/shop/doc/1593776185472315/Turbox-C610_Application-SDK_v1.0.tar.gz
  
   - Unpack the sdk 
       ```
        tar -xzvf Turbox-C610_Application-SDK_v1.0.tar.gz
       ```

   - Execute the below script file it will ask the default target directory, press enter and type 'Y'
       ```
       ./oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh
       ```
   - This complete the environment setup.

### 2. Camera Environment configuration setup on the target device.
   - To setup the camera environment configuration follow the ‘Turbox-C610_Open_Kit_Software_User_Manual_LE1.0_v2.0.pdf’ In given url 
    
 ```
  “https://www.thundercomm.com/app_en/product/1593776185472315” 
    
 ```
    and refer section 2.10.1

### 3. Setting up AWS-iot connect
  
  - To setup the aws-iot-connect refer the below link and inside the iot quick start document in step2, while creating the think object, select device as python instead of js.
```
  https://docs.aws.amazon.com/iot/latest/developerguide/iot-quick-start.html  
```
Save the credentials to the certificates folder in the application directory.

### 4. Compiling the aws-iot-c++ sdk on yocto environment
**Step-1:** In this step clone the git meta-aws repository
        
  ```
  $ git clone https://github.com/aws/meta-aws.git
        
 ```     
**Step-2:**  Copy the aws-iot-device-sdk-cpp-v2 to yocto project  
         
  ```
 $ cp meta-aws-master/recipes-sdk/ <working directory>/poky/meta-aws/recipes-sdk/
 $ cd  <working directory> 
         
 ```

**Step- 3:**  Initialize the build environment

```
$ export  SHELL=/bin/bash
$ source poky/qti-conf/set_bb_env.sh
       
```  

 - Select “qcs610-odk meta-qti-bsp” from the available machines menu. On selecting the machine, the next ncurses menu starts. Select “qti-distro-fullstack-perf meta-qti-bsp” from Available Distributions menu. The shell prompt move to the follow
 
```
<working directory>/build-qti-distro-fullstack-perf/$
```
     
 **Step-4:** Add the the aws-meta path in bblayers.conf file in ‘<working directory>/build-qti-distro-fullstack-perf/conf/bblayers.conf ’ folder

 **Step -5:** run the bitbake command 
  ``` 
   $ bitbake  aws-iot-device-sdk-cpp-v2
  ```
- After running sucesfully this command, you can find the required compiled library as well as include directory folders in path “<build_directory>/tmp-gibc/
sysroots-components/armv7ahf-neon/aws-iot-device-sdk-cpp-v2/usr/”,

- Copy the lib/ directory  to the application directory(‘aws-iot’). 
  ```
    $ cp <build_directory>/tmp-gibc/sysroots-components/armv7ahf-neon/aws-iot-device-sdk-cpp-v2/usr/lib    /home/user/iot_aws/.
  ```

**Step-6:** To compile application code use below bitbake command 
 ```     
   $ bitbake -c compile -f  aws-iot-device-sdk-cpp-v2
 ```
 
 **Step-7:** After running  the bitbake command you can find the git version of source code is available in “<build_directory>/tmp-gibc/work/armv7ahf-neon-oe-linux-gnueabi/aws-iot-device-sdk-cpp-v2/1.6.0-r0/” folder. 
 - Inside the source code, replace the basic_pub_sub folder in path “git/samples/mqtt/basic_pub_sub” with ‘basic_pub_sub’ folder given in our repository. 
 - Execute the bitbake command given in step-6, After the execution, target binary is generated in “build/samples/mqtt/basic_pub_sub/basic_pub_sub”      

 **Step-8:** Rename the binary ‘basic_pub_sub’ to ‘publish’ and copy to the application directory(‘aws-iot’).       
    ```
     $ cp <build_directory>/tmp-gibc/work/armv7ahf-neon-oe-linux-gnueabi/aws-iot-device-sdk-cpp-v2/1.6.0-r0/build/samples/mqtt/basic_pub_sub/basic_pub_sub  /home/user/iot_aws/.
    ```
## Steps to build the camera application: 

- Building the camera application:
**Step-1:** Enter below command to set up the cross compile environment on the host system.
  ```
   $ git clone <source repository>
   $ cd aws-iot 
   $ source /usr/local/oecore-x86_64/environment-setup-armv7ahf-neon-oe-linux-gnueabi
  ```
  
**Step-2:** Build the camera application binary using below command .                                       
   ```
   $ $CC video_record.c video_record.h -o videoaws `pkg-config --cflags --libs gstreamer-1.0`
   ```
   
**Step-3:** Push the camera application binary to the target board with adb command.
  ```
      $ adb root 
      $ adb remount 
      $ adb shell  mount -o remount,rw /
      $ adb forward tcp:8900 tcp:8900
      $ adb push videoaws /data/aws-iot/
  ```

## Step for including aws cert-key files
   - Update the certificate details in publish.py and basic_pub_sub/main.cpp file.
   - push the certificated to target board
   
   ``` 
   $ adb push certificates /data/aws-iot/
   ```
   
## Step for AWS Python IOT SDK Integration 

  -  push the aws-iot-python-sdk, and python source code to target board
  ```
    $ git clone https://github.com/aws/aws-iot-device-sdk-python.git
    $ cd aws-iot-device-sdk-python
    $ adb push  AWSIoTPythonSDK/  /data/aws-iot/
    $ cd ..
    $ adb  publish.py  /data/aws-iot/
  ```
 - push the json library folder to python 3.5 library folder on the target board
 ```
    $ adb push json/  /usr/lib/python3.5/
    $ adb shell
    /# chmod +x /data/aws-iot/videoaws
```

## Step for AWS IOT c++ sdk Integration to target board
  **Note:** Make sure that you have already downloaded the meta-aws repository source code from github and compile the aws-iot-c++ sdk on the yocto environment (steps are given in prerequest of this document)
  
 - push the aws-iot-c++ sdk compiled library to the target board. 
  ```
  $ adb push lib/  /data/aws-iot/
  ```
  - push the c ++ target executable to the target board
  ```      
  $ adb push publish  /data/aws-iot/
  ```
         
## Execute the binary file in the target environment.
            To start the application, run the below commands on the qcs610 board, And also select the command line option as given in the table for different video formats. To stop capturing press ctrl + c on the terminal board. Video will be stored or streamed as per given command line option.
  ```
   /# cd data/aws_iot/
  ```
  - To enable wifi connection
  ```
   /# wpa_supplicant -Dnl80211 -iwlan0 -c /etc/misc/wifi/Wpa_Supplicant.conf -ddddt &
   /# dhcpcd wlan0
 ```
 - To add data/aws-library to the library path 
 ```
    / # export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/aws-iot/lib/
 ```
 - Execute the command
 ```
   /# ./videoaws  {video_format}  {output_path/ip}     {c++/python}
```


| Video format | cmdline-1 | cmdline-2 | cmdline-3 | 
|  :---: | :---: | :---: | :---: |
| 4K Video | 4k | /data/4k.mp4 |  c++/python |
| Hd video | 1080p | /data/1080p.mp4 | c++/python |
| Tcp Streaming | tcp | 127.0.0.1 | c++/python |

**Example:** for playing 4k video
```
   /# ./videoaws 4k /data/video/4k.mp4 c++   
```
- Whenever the camera application starts and stops capturing, the application internally establishes connection to the aws-iot-core and publishes the current status of the camera details in json format to the particular topic. The details include recording format, device details and current action of the camera. The subscriber of this topic will receive the camera  details.  

## To Download video: 
 - Using the adb pull command we can download the video to the host system.
  ```  
    $ adb pull /data/4k.mp4
  ```
- To see the live streaming,  open the new terminal in the host system and enter below command also make sure you have installed the ‘vlc player’ on the host system.
 ```    
    $ adb forward tcp:8900 tcp:8900
    $ vlc -vvv tcp://127.0.0.1:8900
 ```                
 
