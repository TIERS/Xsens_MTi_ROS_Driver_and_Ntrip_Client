
# Xsens MTi ROS Driver and Ntrip Client

This code was based on the official ``xsens_ros_mti_driver`` and tested on MTi-680.
#### Note: the UTC Time, Status Word, Latitude and Longitude, Pvt Data needs to be enabled, in order to get GPGGA data: MT Manager - Device Settings - Output Configuration , select "UTC Time, Status Word, Latitude and Longitude, Pvt Data" and other required data, click "Apply"

Here are the recommended Output Configurations and Device Settings:

![Alt text](MTi-680_Output_Configuration.png)

![Alt text](MTi-680_Device_Settings.png)

## Changes made to the MTi ROS Driver:

 - add ``ntrip_util.h`` and ``ntrip_util.cpp`` under ``src`` folder, to support the /nmea topic
 - add ``xsens_log_handler.h`` and ``xsens_log_handler.cpp`` under ``src`` folder to support the ``should_log`` option in the ``xsens_mti_node.yaml``
 - add ``nmeapublisher.h`` under ``src/messagepublisher`` folder, to send GPGGA message, ``/nmea`` rostopic. The ``/nmea`` messages will be generated by PvtData, but if you have enabled SendLatest Time Synchronization option, you won't be able to get PvtData, even there is trigger, in that case, the ``/nnmea`` messages will be generated by ``packet.UtcTime(), packet.status(), packet.latitudeLongitude()``. The /nmea messages are published at approximately 1Hz.
 - add ``gnssposepublisher.h`` under ``src/messagepublisher`` folder, to send position+orientation in one message, ``/gnss_pose`` rostopic.
 - add ``utctimepublisher.h`` under ``src/messagepublisher`` folder, to send utctime(if available) , ``/imu/utctime`` rostopic. 

change:
 - ``lib/xspublic/xscontroller/iointerface.h``, line 138, change to ``PO_OneStopBIt`` for PO_XsensDefaults.
 - ``lib/xspublic/xscommon/threading.cpp``, line 387 to 408, change the threading behavior, this will be useful for ubuntu 22 OS.

## Ntrip_Client
The Ntrip_client subscribes to the ``/nmea`` rostopic from ``xsens_ros_mti_driver``, and wait until it gets data for maximum 300 sec, it will send GPGGA to the Ntrip Caster(Server) every 10 seconds.

User needs to change the ``ntrip.launch`` for their own credentials/servers/mountpoint. 

## How to Install:
install dependency:
```
sudo apt install ros-[ROSDISTRIBUTION]-nmea-msgs
sudo apt install ros-[ROSDISTRIBUTION]-mavros-msgs
```
for example for ROS Melodic:
```
sudo apt install ros-melodic-nmea-msgs
sudo apt install ros-melodic-mavros-msgs
```

clone the source file to your ``catkin_ws``, and run the code below:
```
cd ~/catkin_ws
pushd src/xsens_ros_mti_driver/lib/xspublic && make && popd
catkin_make
```
Source the ``/devel/setup.bash`` file inside your catkin workspace
```
source ./devel/setup.bash
```
or 

add it into rules:
```
sudo nano ~/.bashrc
```
At the end of the file, add the following line:
```
source /[PATH_TO_Your_catkin_ws]/devel/setup.bash
```
save the file, exit.

## How to Use:
change the credentials/servers/mountpoint in ``src/ntrip/launch/ntrip.launch`` to your own one.


open two terminals:
```
roslaunch xsens_mti_driver xsens_mti_node.launch
```
or with the 3D display rviz:
```
roslaunch xsens_mti_driver display.launch
```
and then
```
roslaunch ntrip ntrip.launch
```

## How to confirm your RTK Status

you could check ``rostopic echo /rtcm``, there should be HEX RTCM data coming,

or ``rostopic echo /status`` to check the RTK Fix type, it should be Floating or Fix.


## ROS Topics

| topic                    | Message Type                    | Message Contents                                                                                                                              | Data Output Rate<br>(Depending on Model and OutputConfigurations at MT Manager) |
| ------------------------ | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| filter/free_acceleration | geometry_msgs/Vector3Stamped    | free acceleration from filter, which is the acceleration in the local earth coordinate system (L) from which<br>the local gravity is deducted | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| filter/positionlla       | geometry_msgs/Vector3Stamped    | filtered position output in latitude (x), longitude (y) and altitude (z) as Vector3, in WGS84 datum                                           | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| filter/quaternion        | geometry_msgs/QuaternionStamped | quaternion from filter                                                                                                                        | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| filter/twist             | geometry_msgs/TwistStamped      | velocity and angular velocity                                                                                                                 | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| filter/velocity          | geometry_msgs/Vector3Stamped    | filtered velocity output as Vector3                                                                                                           | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| gnss                     | sensor_msgs/NavSatFix           | raw 4 Hz latitude, longitude, altitude and status data from GNSS receiver                                                                     | 4Hz                                                                             |
| gnss_pose                | geometry_msgs/PoseStamped       | filtered position output in latitude (x), longitude (y) and altitude (z) as Vector3 in WGS84 datum, and quaternion from filter                | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/acceleration         | geometry_msgs/Vector3Stamped    | calibrated acceleration                                                                                                                       | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/angular_velocity     | geometry_msgs/Vector3Stamped    | calibrated angular velocity                                                                                                                   | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/data                 | sensor_msgs/Imu                 | quaternion, calibrated angular velocity and acceleration                                                                                      | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/dq                   | geometry_msgs/QuaternionStamped | integrated angular velocity from sensor (in quaternion representation)                                                                        | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/dv                   | geometry_msgs/Vector3Stamped    | integrated acceleration from sensor                                                                                                           | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| imu/mag                  | geometry_msgs/Vector3Stamped    | calibrated magnetic field                                                                                                                     | 1-100Hz                                                                         |
| imu/time_ref             | sensor_msgs/TimeReference       | SampleTimeFine timestamp from device                                                                                                          | depending on packet                                                             |
| imu/utctime              | sensor_msgs/TimeReference       | UTC Time from the device                                                                                                                      | depending on packet                                                             |
| nmea                     | nmea_msgs/Sentence              | 1Hz GPGGA data from GNSS receiver PVTData(if available), otherwise from filtered position, utctime, status packet                             | 1Hz                                                                             |
| pressure                 | sensor_msgs/FluidPressure       | barometric pressure from device                                                                                                               | 1-100Hz                                                                         |
| status                   | diagnostic_msgs/DiagnosticArray | statusWord, 32bit                                                                                                                             | depending on packet                                                             |
| temperature              | sensor_msgs/Temperature         | temperature from device                                                                                                                       | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |
| tf                       | geometry_msgs/TransformStamped  | transformed orientation                                                                                                                       | 1-400Hz(MTi-600 and MTi-100 series), 1-100Hz(MTi-1 series)                      |

Please refer to [MTi Family Reference Manual](https://mtidocs.xsens.com/mti-system-overview) for detailed definition of data. 
