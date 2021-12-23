# ROS environment for testing Ouster OS1-32
- Docker container for testing SLAM packages with Ouster Lidar
- Currently built upon the ROS melodic container for Livox MID70
- Currently tested packages:
    - FAST-LIO
    - LIO-SAM

## Requirements
- PTPD (sudo apt install ptpd), Docker, NVIDIA Container Toolkit  

## How to use
1. `docker pull eehantiming/ouster:{tagnumber}`  
2. Edit run.sh with the appropriate tag number, and (optional) any volumes that you want to mount 
3. Connect the LIDAR via ethernet and set IPv4 to 'link-local only'. Check your connection with `ping os-{serial number}.local` and note down the IP address. Note: you must connect before starting the container
4. If required, connect IMU via USB and start clock master with `sudo ptpd -M -i eth0 -C` (check ifconfig and change eth0)
5. `./run.sh`. There are currently 2 workspaces, ros_ws and ouster_ws. `source {ws}/devel/setup.bash` as required  
6. Use `docker cp {file} ouster:/root` to copy rosbag or other files in. You can also use this to copy files out  

### [Ouster ROS driver](https://github.com/ouster-lidar/ouster_example)
- This package is required when working with live LIDAR data. Publishes to /os_cloud_node/imu and os_cloud_node/points
- Create a .json and pass the path into the ros arg (example below)  
`source ouster_ws/devel/setup.bash && roslaunch ouster_ros ouster.launch sensor_hostname:=169.254.234.124 metadata:=/root/ouster_ws/src/ouster_example/test.json timestamp_mode:=TIME_FROM_PTP_1588`
- Add `viz:=true` to open rviz. Rviz usually opens with any mapping package that you are using. You may find the other [ros args here](https://github.com/ouster-lidar/ouster_example/blob/master/ouster_ros/ouster.launch)  
- Time sync via timestamp_mode is required so that lidar points timestamp matches those from an external IMU. It is not required if only testing LIDAR + Internal IMU.  

### [FAST-LIO](https://github.com/DinoHub/FAST_LIO)  
- FAST-LIO contains a config and a launch with support for ouster + internal imu.
    1. Update extrinsic_T and other configs in config/ouster64.yaml
    2. `source ros_ws/devel/setup.bash && roslaunch fast_lio mapping_ouster64.launch` 
    3. `source ouster_ws/devel/setup.bash && roslaunch ouster_ros ouster.launch sensor_hostname:=169.254.234.124 metadata:=/root/ouster_ws/src/ouster_example/test.json`

- To use with external imu:
    1. Ensure that ptpd is started
    2. Find imu tty with `dmesg | grep tty`. Edit param ACM0 or ACM1 in launch/mapping_ouster32.launch if required.
    3. `source ros_ws/devel/setup.bash && roslaunch fast_lio mapping_ouster32.launch`  
    4. Turn on IMU stream with `rosservice call mavros/set_stream_rate`. message rate 200 and 'true'  
    5. `source ouster_ws/devel/setup.bash && roslaunch ouster_ros ouster.launch sensor_hostname:=169.254.234.124 metadata:=/root/ouster_ws/src/ouster_example/test.json timestamp_mode:=TIME_FROM_PTP_1588`
    6. If mapping fails, check points timestamp (imu and points have the same timestamp but easier to read) `rostopic echo /os_cloud_node/imu` against `rostopic echo /mavros/imu/data`

### [LIO-SAM](https://github.com/TixiaoShan/LIO-SAM)
- LIO-SAM does not work with ouster internal 6-axis IMU.    
    1. Ensure that ptpd is started
    2. Find imu tty with `dmesg | grep tty`. Edit param ACM0 or ACM1 in launch/run.launch if required.
    3. `source ros_ws/devel/setup.bash && roslaunch lio_sam run.launch`  
    4. Turn on IMU stream with `rosservice call mavros/set_stream_rate`. message rate 200 and 'true'  
    5. `source ouster_ws/devel/setup.bash && roslaunch ouster_ros ouster.launch sensor_hostname:=169.254.234.124 metadata:=/root/ouster_ws/src/ouster_example/test.json timestamp_mode:=TIME_FROM_PTP_1588`
    6. If mapping fails, check points timestamp (imu and points have the same timestamp but easier to read) `rostopic echo /os_cloud_node/imu` against `rostopic echo /mavros/imu/data`

### [loam_velodyne](https://github.com/DinoHub/loam_velodyne)
- Added support for ouster. To be tested with rosbag/lidar

## Image versions
Tag | Description (Packages added on top of all previous versions)
---|---
1.0 |  livox driver and packages + <br> ouster driver <br> loam_velodyne <br>