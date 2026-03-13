# Lab: Camera Sensor in Gazebo with ROS 2 Jazzy

# 1. Install Gazebo Models for World
## Clone models
``` bash
git clone https://github.com/osrf/gazebo_models ~/gazebo_models
```
## Add path to bashrc
``` bash
echo "export GZ_SIM_RESOURCE_PATH=$HOME/gazebo_models" >> ~/.bashrc
source ~/.bashrc
```

---

# 2. Launch Gazebo with a World

Download the **house world** or use any Gazebo world.

Launch Gazebo with ROS support:

```bash
ros2 launch ros_gz_sim gz_sim.launch.py gz_args:=house.world
```
---

# 3. Create the Robot URDF

Create file:

```
camera_robot.urdf
```

Paste the following **URDF version (converted from your Xacro)**.

```xml
<?xml version="1.0"?>

<robot name="camera_robot">

  <!-- Base Link -->
  <link name="base_link">

    <inertial>
      <origin xyz="0 0 0"/>
      <mass value="1.0"/>
      <inertia ixx="0.1" iyy="0.1" izz="0.1"
               ixy="0" ixz="0" iyz="0"/>
    </inertial>

    <visual>
      <geometry>
        <box size="0.2 0.2 0.1"/>
      </geometry>
    </visual>

    <collision>
      <geometry>
        <box size="0.2 0.2 0.1"/>
      </geometry>
    </collision>

  </link>

  <!-- Camera Link -->

  <link name="camera_link">

    <visual>
      <geometry>
        <box size="0.05 0.05 0.05"/>
      </geometry>
    </visual>

  </link>

  <!-- Fixed Joint -->

  <joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.1 0 0.05"/>
  </joint>

  <!-- Gazebo Camera Sensor -->

  <gazebo reference="camera_link">

    <sensor type="camera" name="camera">
      <always_on>true</always_on>
      <update_rate>30</update_rate>
      <visualize>true</visualize>

      <camera>
        <horizontal_fov>1.396</horizontal_fov>

        <image>
          <width>640</width>
          <height>480</height>
          <format>R8G8B8</format>
        </image>

        <clip>
          <near>0.1</near>
          <far>100</far>
        </clip>

      </camera>

    </sensor>

  </gazebo>

  <gazebo>
    <plugin name="gz::sim::systems::Sensors" filename="gz-sim-sensors-system">
      <render_engine>orge2</render_engine>
    </plugin>
  </gazebo>

</robot>
```

---

# 4. Spawn Robot in Gazebo (ROS 2 Jazzy)

Open a **new terminal**.

Source ROS:

Spawn robot:

```bash
ros2 run ros_gz_sim create \
-file camera_robot.urdf \
-name camera_bot \
-z 1
```

This will:

* Load the URDF
* Spawn robot in Gazebo
* Start the camera sensor

You should see a **small cube robot with a camera mounted on top**.

## gazebo check
``` bash
gz topic -l
```
you should see `<something>/camera/camera/image`

>[!note]
>Change the `<something>` to complete topic namge

---

# 5. Check Camera Topics

## ros bridge
``` bash
ros2 run ros_gz_bridge parameter_bridge \
<something>/camera/image@gz.msgs.Image \
--ros-args -r <something>/camera/image:=/camera/image_raw
```

List topics:

```bash
ros2 topic list
```

Expected output:

```
/camera/image_raw
```

Check camera info:

```bash
ros2 topic echo /camera/image_raw --once
```

---

# 6. View Camera Stream

Open RViz2.

```bash
rviz2
```

Steps inside RViz:

1. Click **Add**
2. Select **Image**
3. Change topic to

```
/camera/image_raw
```

You should see the **live camera feed from Gazebo**.

---

# 7. Check Camera Frame (Optional)

You can inspect the TF frame.

```bash
ros2 run tf2_ros tf2_echo base_link camera_link
```

This shows the **transform between robot base and camera**.

---

# Expected Topics from Camera

| Topic                 | Type                   | Description             |
| --------------------- | ---------------------- | ----------------------- |
| `/camera/image_raw`   | sensor_msgs/Image      | Raw camera image        |
| `/camera/camera_info` | sensor_msgs/CameraInfo | Camera calibration data |
| `/clock`              | rosgraph_msgs/Clock    | Simulation time         |

---

# What Changed for ROS 2 Jazzy

Compared to older tutorials:

| Old                | Jazzy                  |
| ------------------ | ---------------------- |
| ROS1 `spawn_model` | ROS2 `spawn_entity.py` |
| ROS1 `rviz`        | ROS2 `rviz2`           |
| Gazebo plugins     | `gazebo_ros_pkgs`      |
| ROS topics         | DDS based              |

---

💡 **Tip for students**

Run this to visualize images quickly:

```bash
ros2 run rqt_image_view rqt_image_view
```

Then select:

```
/camera/image_raw
```

---

If you want, I can also show you the **proper ROS2 Jazzy project structure** for this lab (with `colcon build`, `launch.py`, and packages). That’s usually the **next step after this assignment** and makes it much cleaner.
