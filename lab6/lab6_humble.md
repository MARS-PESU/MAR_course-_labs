# turtlebot3 simulations

## Install Dependencies

    $ sudo apt install ros-humble-gazebo-*

    Install Cartographer
    [Remote PC]

    $ sudo apt install ros-humble-cartographer
    $ sudo apt install ros-humble-cartographer-ros

    Install Navigation2
    [Remote PC]

    $ sudo apt install ros-humble-navigation2
    $ sudo apt install ros-humble-nav2-bringup

Install TurtleBot3 Packages

Install the required TurtleBot3 Packages.

[Remote PC]

$ source /opt/ros/humble/setup.bash
$ mkdir -p ~/turtlebot3_ws/src
$ cd ~/turtlebot3_ws/src/
$ git clone -b humble https://github.com/ROBOTIS-GIT/DynamixelSDK.git
$ git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
$ git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git
$ sudo apt install python3-colcon-common-extensions
$ cd ~/turtlebot3_ws
$ colcon build --symlink-install
$ echo 'source ~/turtlebot3_ws/install/setup.bash' >> ~/.bashrc
$ source ~/.bashrc

Environment Configuration

    Setup your ROS environment for the Remote PC.
    [Remote PC]

    $ echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc
    $ echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
    $ source ~/.bashrc


## Follow gazebo simulation 

### Install Simulation Package

The **TurtleBot3 Simulation Package** requires `turtlebot3` and `turtlebot3_msgs` packages. Without these prerequisite packages, the Simulation cannot be launched.  
Please follow the [PC Setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/) instructions if you did not install required packages and dependent packages.

```
$ cd ~/turtlebot3_ws/src/
$ git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ cd ~/turtlebot3_ws && colcon build --symlink-install
```

### Launch Simulation World]
Three simulation environments are prepared for TurtleBot3. Please select one of these environments to launch Gazebo.

**Please make sure to completely terminate any other Simulation world before launching a new world.**

1. Empty World  
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_empty_world.png)
    
    ```
    $ export TURTLEBOT3_MODEL=burger
    $ ros2 launch turtlebot3_gazebo empty_world.launch.py
    ```
    
2. TurtleBot3 World  
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/gazebo_world.png)
    
    ```
    $ export TURTLEBOT3_MODEL=waffle
    $ ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
    ```
    
3. TurtleBot3 House  
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_house.png)
    
    ```
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py
    ```
    

**NOTE**: If TurtleBot3 House is launched for the first time, downloading the map may take more than a few minutes depending on network status.

### [Operate TurtleBot3](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#operate-turtlebot3)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#operate-turtlebot3)

In order to teleoperate the TurtleBot3 with a keyboard, launch the teleoperation node with the command below in a new terminal window.

```
$ ros2 run turtlebot3_teleop teleop_keyboard
```

 ![](https://emanual.robotis.com/assets/images/icon_unfold.png) Read more about **How to run Autonomous Collision Avoidance**

A simple collision avoidance node which keeps a safe distance from obstacles and makes turns to avoid collisions is provided with the TurtleBot3 simulation packages.  
In order to autonomously drive a TurtleBot3 in the **TurtleBot3 world**, please follow the instructions below.

1. Terminate the turtlebot3_teleop_key node by entering `Ctrl` + `C` in the terminal running the teleop node.
    
2. Enter the command below in the terminal.
    
    ```
    $ ros2 run turtlebot3_gazebo turtlebot3_drive
    ```
 ![](https://emanual.robotis.com/assets/images/icon_unfold.png) 

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_gazebo_rviz.png)


## SLAM Simulation

With SLAM in the Gazebo simulator, you can select or create various environments and robot models in a virtual world. Other than the preparation of the simulation environment instead of bringing up the robot, SLAM Simulation is pretty similar to the operation of [SLAM](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam/#slam) on the actual TurtleBot3.

The following instructions require prerequisites from the previous section, so please review the [Simulation](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/) section first.

### Launch Simulation World[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#launch-simulation-world)

Three Gazebo environments are prepared, but for creating a map with SLAM, it is recommended to use either **TurtleBot3 World** or **TurtleBot3 House**.  
Use one of the following commands to load the Gazebo environment.

In this tutorial, TurtleBot3 World will be used.  
Specify your TurtleBot3 model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

 ![](https://emanual.robotis.com/assets/images/icon_unfold.png) Read more about **How to load TurtleBot3 House** Specify your TurtleBot3 model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py
```

### Run SLAM Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#run-slam-node)

Open a new terminal on the Remote PC with `Ctrl` + `Alt` + `T` and run the SLAM node. Cartographer SLAM method is used by default. Specify your TurtleBot3 model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_cartographer cartographer.launch.py use_sim_time:=True
```

### Run Teleoperation Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#run-teleoperation-node)

Open a new terminal on the Remote PC with `Ctrl` + `Alt` + `T` and run the teleoperation node from the Remote PC. Specify your TurtleBot3 model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 run turtlebot3_teleop teleop_keyboard

 Control Your TurtleBot3!
 ---------------------------
 Moving around:
        w
   a    s    d
        x

 w/x : increase/decrease linear velocity
 a/d : increase/decrease angular velocity
 space key, s : force stop

 CTRL-C to quit
```

### Save Map[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#save-map)

When the map is has been created, open a new terminal on the Remote PC with `Ctrl` + `Alt` + `T` and save the map.

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/virtual_slam.png)

```
$ ros2 run nav2_map_server map_saver_cli -f ~/map
```

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/map.png)

## Navigation Simulation
Just like with SLAM in the Gazebo simulator, you can select or create various environments and robot models in the virtual Navigation world. However, a complete map has to be prepared before running Navigation. Other than the preparation of a simulation environment instead of bringing up the robot, Navigation Simulation is pretty similar to that of real-world TurtleBot3 [Navigation](https://emanual.robotis.com/docs/en/platform/turtlebot3/navigation/#navigation).

### Launch Simulation World[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#launch-simulation-world)

Terminate `Ctrl` + `C` all applications that were launched in the previous sections.

In the previous [SLAM](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam/#slam) section, TurtleBot3 World was used to create a map. The same Gazebo environment will be used for Navigation.

Specify your TurtleBot model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

 ![](https://emanual.robotis.com/assets/images/icon_unfold.png) Read more about **How to load TurtleBot3 House**

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py
```

### Run Navigation Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#run-navigation-node)

Open a new terminal from Remote PC with `Ctrl` + `Alt` + `T` and run the Navigation2 node.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=True map:=$HOME/map.yaml
```

### [Estimate Initial Pose](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#estimate-initial-pose)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#estimate-initial-pose)

**Initial Pose Estimation** must be performed before running Navigation as this process initializes the AMCL parameters that are critical for accurate Navigation. TurtleBot3 has to be correctly located on the map with the LDS sensor data that neatly overlaps the displayed map.

1. Click the `2D Pose Estimate` button in the RViz2 menu.
2. Click on the map where the actual robot is located and drag the large green arrow toward the direction where the robot is facing.
3. Repeat step 1 and 2 until the LDS sensor data is overlaid on the saved map. ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tb3_navigation2_rviz_01.png)
4. Launch keyboard teleoperation node to precisely locate the robot on the map.
    
    ```
    $ ros2 run turtlebot3_teleop teleop_keyboard
    ```
    
5. Move the robot back and forth a bit to collect the surrounding environment information and narrow down the estimated location of the TurtleBot3 on the map which is displayed with tiny green arrows.  
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tb3_amcl_particle_01.png) ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tb3_amcl_particle_02.png)
6. Terminate the keyboard teleoperation node by entering `Ctrl` + `C` to the teleop node terminal in order to prevent different **cmd_vel** values are published from multiple nodes during Navigation.

### [Set Navigation Goal](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#set-navigation-goal)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#set-navigation-goal)

1. Click the `Navigation2 Goal` button in the RViz2 menu.
2. Click on the map to set the destination of the robot and drag the green arrow toward the direction where the robot will be facing.
    - This green arrow is a marker that can specify the destination of the robot.
    - The root of the arrow is `x`, `y` coordinate of the destination, and the angle `θ` is determined by the orientation of the arrow.
    - As soon as x, y, θ are set, TurtleBot3 will start moving to the destination immediately. ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tb3_navigation2_rviz_02.png)


## Now finally to understand the working of camera and encoders:

# TO be updated

