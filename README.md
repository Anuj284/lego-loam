# LeGO-LOAM

This repository contains code for a lightweight and ground optimized lidar odometry and mapping (LeGO-LOAM) system for Autonomous Vehicles. The system takes in point cloud from a Velodyne VLP-16 Lidar (palced horizontal) and IMU data as inputs. It outputs 6D pose estimation in real-time. 

<p align='center'>
    <img src="/LeGO-LOAM/launch/demo.gif" alt="drawing" width="800"/>
</p>

## Docker
The easiest way to run this package is by using the provided docker image. **Needs NVIDIA GPU**.

  ### Dependencies for docker

  - Install **docker-ce** using the instructions given below.
    ```sh
    sudo apt-get update

    # to uninstall older versions of docker
    sudo apt-get remove docker docker-engine docker.io

    sudo apt-get update
	
    sudo apt-get install \
         apt-transport-https \
         ca-certificates \
         curl \
         software-properties-common
 	
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 	
    sudo apt-key fingerprint 0EBFCD88
    #Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching 	  	   the last 8 characters of the fingerprint.
 	
    sudo add-apt-repository \
         "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) \
         stable"
	
    sudo apt-get update
	
    sudo apt-get install docker-ce
    ```
  - Install [**nvidia-docker2**](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)) plugin. Check the link for its dependencies.
    ```sh
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

    sudo apt-get update
    # purge older version of nvidia-docker
    sudo apt-get purge nvidia-docker

    sudo apt-get install nvidia-docker2
    sudo pkill -SIGHUP dockerd
    ```
  ### Run container
  Terminal 1
  ```sh
  git clone https://github.com/ayushgargdroid/lego-loam.git
  roscore
  ```
  Terminal 2
  ```sh
  cd lego-loam/docker_files/
  ./run_script.bash
  ```
  Terminal 3 (Play the morning rosbag)
  ```sh
  rosbag play morning_stereo_rgb_ir_lidar_gps.bag --clock --topics /ns1/velodyne_points /vehicle/gps/fix /camera_array/cam0/camera_info /camera_array/cam0/image_raw /imu/imu
  ```
  Sit back and relax. Your welcome!

  If you are having issues with **Docker permissions**, please follow this [link](https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket).

## Dependency (if Docker did not work for you)

We assume you have ROS Melodic installed and the environment setup.

- [ROS](http://wiki.ros.org/ROS/Installation) 
- libpcl>1.8 
  ```sh
  sudo apt-get update
  sudo apt-get install libpcl-*1.8
  ```
- [gtsam](https://github.com/borglab/gtsam/releases) (Georgia Tech Smoothing and Mapping library, 4.0.0-alpha2)
  ```sh
  wget -O ~/Downloads/gtsam.zip https://github.com/borglab/gtsam/archive/4.0.0-alpha2.zip
  cd ~/Downloads/ && unzip gtsam.zip -d ~/Downloads/
  cd ~/Downloads/gtsam-4.0.0-alpha2/
  mkdir build && cd build
  cmake -D GTSAM_WITH_EIGEN_MKL=OFF -D CMAKE_CXX_FLAGS="-w" ..
  sudo make install -j8
  ```
- [GeographicLib](https://sourceforge.net/projects/geographiclib/files/distrib/) (ver 1.50.1) We have a copy of it in the repo directory.
  ```sh
  cd ~/Downloads
  wget https://github.com/ayushgargdroid/lego-loam/blob/master/GeographicLib-1.50.1.tar.gz
  tar -xf GeographicLib-1.50.1.tar.gz
  cd GeographicLib-1.50.1/ && mkdir build
  cmake ..
  sudo make install -j8
  ```
- ros-melodic-tf2-sensor-msgs
  ```sh
  sudo apt-get install ros-melodic-tf2-sensor-msgs
  ```
- [imu_transformer](https://github.com/ayushgargdroid/imu_transformer) 
  ```sh
  cd ~/catkin_ws/src
  git clone https://github.com/ayushgargdroid/imu_transformer.git
  cd ..
  catkin_make
  source devel/setup.bash
  ```
- morning_stereo_rgb_ir_lidar_gps.bag dataset

## Compile

You can use the following commands to download and compile the package.

```sh
cd ~/catkin_ws/src
git clone https://github.com/ayushgargdroid/lego-loam.git
cd ..
catkin_make -j1
source devel/setup.bash
```
When you compile the code for the first time, you need to add "-j1" behind "catkin_make" for generating some message types. "-j1" is not needed for future compiling.

## Run the package

1. Run the launch file:
```sh
roslaunch lego_loam run.launch
```
Notes: The parameter "/use_sim_time" is set to "true" for simulation, "false" to real robot usage.

2. Play existing bag files **strictly like this**: 
```sh
rosbag play morning_stereo_rgb_ir_lidar_gps.bag --clock --topics /ns1/velodyne_points /vehicle/gps/fix /camera_array/cam0/camera_info /camera_array/cam0/image_raw /imu/imu
```

```
@inproceedings{behley2019iccv,
  author    = {J. Behley and M. Garbade and A. Milioto and J. Quenzel and S. Behnke and C. Stachniss and J. Gall},
  title     = {{SemanticKITTI: A Dataset for Semantic Scene Understanding of LiDAR Sequences}},
  booktitle = {Proc. of the IEEE/CVF International Conf.~on Computer Vision (ICCV)},
  year      = {2019}
}
```