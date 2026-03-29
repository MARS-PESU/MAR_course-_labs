# Turtle bot 3 and navigation


## Install Dependent ROS 2 Packages
1. Open the terminal with `Ctrl`+`Alt`+`T` on the **Remote PC**.
2. Install Gazebo Sim  

    
    ```
    $ sudo apt-get update
    $ sudo apt-get install curl lsb-release gnupg
    $ sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
    $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
    $ sudo apt-get update
    $ sudo apt-get install gz-harmonic
    ```
    
3. Install Cartographer  

    
    ```
    $ sudo apt install ros-jazzy-cartographer
    $ sudo apt install ros-jazzy-cartographer-ros
    ```
    
4. Install Navigation2  

    
    ```
    $ sudo apt install ros-jazzy-navigation2
    $ sudo apt install ros-jazzy-nav2-bringup
    ```
    

### [Install TurtleBot3 Packages](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#install-turtlebot3-packages)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#install-turtlebot3-packages-1)

Install the required TurtleBot3 Packages.



```
$ source /opt/ros/jazzy/setup.bash
$ mkdir -p ~/turtlebot3_ws/src
$ cd ~/turtlebot3_ws/src/
$ git clone -b jazzy https://github.com/ROBOTIS-GIT/DynamixelSDK.git
$ git clone -b jazzy https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
$ git clone -b jazzy https://github.com/ROBOTIS-GIT/turtlebot3.git
$ sudo apt install python3-colcon-common-extensions
$ cd ~/turtlebot3_ws
$ colcon build --symlink-install
$ echo 'source ~/turtlebot3_ws/install/setup.bash' >> ~/.bashrc
$ source ~/.bashrc
```

### [Environment Configuration](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#environment-configuration)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#environment-configuration-1)

1. Setup your ROS environment for the Remote PC.  

    
    ```
    $ echo 'export ROS_DOMAIN_ID=30 #TURTLEBOT3' >> ~/.bashrc
    $ echo 'source /opt/ros/jazzy/setup.bash' >> ~/.bashrc
    $ source ~/.bashrc
    ```

## Install Simulation Package
The **TurtleBot3 Simulation Package** requires `turtlebot3` and `turtlebot3_msgs` packages. Without these prerequisite packages, the Simulation cannot be launched.  
Please follow the [PC Setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/) instructions if you did not install required packages and dependent packages.

```
$ cd ~/turtlebot3_ws/src/
$ git clone -b jazzy https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ cd ~/turtlebot3_ws && colcon build --symlink-install
```

### [Launch Simulation World](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#launch-simulation-world)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#launch-simulation-world-1)

Three simulation environments are prepared for TurtleBot3. Please select one of these environments to launch Gazebo.

**Please make sure to completely terminate any other Simulation world before launching a new world.**

1. Empty World
    
    ```
    $ export TURTLEBOT3_MODEL=burger
    $ ros2 launch turtlebot3_gazebo empty_world.launch.py
    ```
    
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_empty_world_sim.png)
    
2. TurtleBot3 World
    
    ```
    $ export TURTLEBOT3_MODEL=waffle
    $ ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
    ```
    
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_world_sim.png)
    
3. TurtleBot3 House
    
    ```
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py
    ```
    
    ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_house_sim.png)
    

**NOTE**: If TurtleBot3 House is launched for the first time, downloading the map may take more than a few minutes depending on network status.

### [Operate TurtleBot3](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#operate-turtlebot3)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#operate-turtlebot3-1)

In order to teleoperate the TurtleBot3 with a keyboard, launch the teleoperation node with the command below in a new terminal window.

```
$ ros2 run turtlebot3_teleop teleop_keyboard
```

 ![](https://emanual.robotis.com/assets/images/icon_unfold.png) 

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_gazebo_rviz.png)

## SLAM Simulation
With SLAM in the Gazebo simulator, you can select or create various environments and robot models in a virtual world. Other than the preparation of the simulation environment instead of bringing up the robot, SLAM Simulation is pretty similar to the operation of [SLAM](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam/#slam) on the actual TurtleBot3.

The following instructions require prerequisites from the previous section, so please review the [Simulation](https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/) section first.

### Launch Simulation World[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#launch-simulation-world-1)

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

### Run SLAM Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#run-slam-node-1)

Open a new terminal on the Remote PC with `Ctrl` + `Alt` + `T` and run the SLAM node. Cartographer SLAM method is used by default. Specify your TurtleBot3 model (`burger`, `waffle`, `waffle_pi`) using the `TURTLEBOT3_MODEL` parameter.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_cartographer cartographer.launch.py use_sim_time:=True
```

### Run Teleoperation Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#run-teleoperation-node-1)

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

### Save Map[](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam_simulation/#save-map-1)

When the map is has been created, open a new terminal on the Remote PC with `Ctrl` + `Alt` + `T` and save the map.

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/virtual_slam_sim.png)

```
$ ros2 run nav2_map_server map_saver_cli -f ~/map
```

![](https://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/map.png)

## Navigation Simulation

Just like with SLAM in the Gazebo simulator, you can select or create various environments and robot models in the virtual Navigation world. However, a complete map has to be prepared before running Navigation. Other than the preparation of a simulation environment instead of bringing up the robot, Navigation Simulation is pretty similar to that of real-world TurtleBot3 [Navigation](https://emanual.robotis.com/docs/en/platform/turtlebot3/navigation/#navigation).

### Launch Simulation World[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#launch-simulation-world-1)

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

### Run Navigation Node[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#run-navigation-node-1)

Open a new terminal from Remote PC with `Ctrl` + `Alt` + `T` and run the Navigation2 node.

```
$ export TURTLEBOT3_MODEL=burger
$ ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=True map:=$HOME/map.yaml
```

### [Estimate Initial Pose](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#estimate-initial-pose)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#estimate-initial-pose-1)

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

### [Set Navigation Goal](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#set-navigation-goal)[](https://emanual.robotis.com/docs/en/platform/turtlebot3/nav_simulation/#set-navigation-goal-1)

1. Click the `Navigation2 Goal` button in the RViz2 menu.
2. Click on the map to set the destination of the robot and drag the green arrow toward the direction where the robot will be facing.
    - This green arrow is a marker that can specify the destination of the robot.
    - The root of the arrow is `x`, `y` coordinate of the destination, and the angle `θ` is determined by the orientation of the arrow.
    - As soon as x, y, θ are set, TurtleBot3 will start moving to the destination immediately. ![](https://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tb3_navigation2_rviz_02.png)

- - -

before starting the next session plese close all your terminals open as of now.

- - -

## Now finally to understand the working of camera and encoders:

### Launch TurtleBot3 Waffle Pi in Gazebo

Set your model:

``` bash
export TURTLEBOT3_MODEL=waffle_pi
```

Launch simulation:

```bash
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

And the navigation too in a different terminal

```bash
export TURTLEBOT3_MODEL=waffle_pi
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=True map:=$HOME/map.yaml
```

You should now see:

- Robot spawned in Gazebo
- Topics being published (camera, odom, tf)


#### topic list

lets listout topics
``` bash
ros2 topic list
```
<img width="854" height="616" alt="Screenshot from 2026-03-29 19-43-54" src="https://github.com/user-attachments/assets/21cbbb73-9002-489a-ac1d-586aae6fabad" />

---

### Understanding Encoders (Odometry)

In simulation, **wheel encoders are abstracted** into odometry data.

#### Key Topic

/odom

Check it:

```
ros2 topic echo --once /odom
```

#### What You’ll See

- `pose.pose.position` → x, y position
- `pose.pose.orientation` → quaternion
- `twist.twist.linear` → linear velocity
- `twist.twist.angular` → angular velocity

#### Concept

Encoders → wheel rotation → robot displacement → odometry

Gazebo computes this using differential drive plugin.
Now lets visualize it in RVIZ


---

### 4. Visualizing Odometry

Run RViz:

```
rviz2
```

Add:

- **Odometry display**
- **TF**
<img width="520" height="715" alt="Screenshot from 2026-03-29 19-45-12" src="https://github.com/user-attachments/assets/b7398ede-00f4-41fc-ab5a-15662a2142df" />
<img width="1220" height="901" alt="Screenshot from 2026-03-29 14-37-04" src="https://github.com/user-attachments/assets/4f4f32c5-e6dc-4c99-bc63-7add4c8143a7" />

#NOTE: if you see a yellowish screen open the left panel > open the settings for odom and switch of covariance

You’ll see:

- Robot moving
- Trajectory updating

now finally start teleop_keyboard and see how the arrow changes with the incoming messages.
<img width="854" height="616" alt="Screenshot from 2026-03-29 19-45-52" src="https://github.com/user-attachments/assets/56a6f39e-5f3c-462b-9332-6af86b749f63" />

<img width="1135" height="631" alt="Screenshot from 2026-03-29 19-49-24" src="https://github.com/user-attachments/assets/aa4fb85d-9f08-4425-b088-3c79cc6a10a7" />


---

### Using the Camera

The Waffle Pi has a simulated RGB camera.

#### Key Topics

/camera/image_raw  
/camera/camera_info



---

#### View Camera Feed

ros2 run rqt_image_view rqt_image_view

Select:

```
/camera/image_raw
```

you can also view it in RVIZ2

---

### Camera Data Structure

Each image message is:

- Type: `sensor_msgs/msg/Image`
- Fields:
    - height, width
    - encoding (e.g. `rgb8`)
    - data (pixel array)  
   # To be updated
