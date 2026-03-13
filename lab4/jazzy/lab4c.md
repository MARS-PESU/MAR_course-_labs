# Lab: Camera Sensor in Gazebo with ROS 2 Jazzy

## 1. Install Required Packages (ROS 2 Jazzy)

Make sure the Gazebo ROS integration packages are installed.

```bash
sudo apt update
sudo apt install \
ros-jazzy-gazebo-ros-pkgs \
ros-jazzy-gazebo-plugins \
ros-jazzy-rviz2
```

Source ROS:

```bash
source /opt/ros/jazzy/setup.bash
```

---

# 2. Launch Gazebo with a World

Download the **house world** or use any Gazebo world.

Launch Gazebo with ROS support:

```bash
gz sim house.world -s libgazebo_ros_factory.so
```

Explanation:

| Part                       | Meaning                                   |
| -------------------------- | ----------------------------------------- |
| `gazebo`                   | Starts the Gazebo simulator               |
| `house.world`              | Loads the environment                     |
| `libgazebo_ros_factory.so` | Enables ROS 2 to spawn robots dynamically |

First launch may take time because Gazebo downloads models.

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

      <update_rate>30</update_rate>

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

      <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
        <camera_name>camera</camera_name>
        <frame_name>camera_link</frame_name>
      </plugin>

    </sensor>

  </gazebo>

</robot>
```

---

# 4. Spawn Robot in Gazebo (ROS 2 Jazzy)

Open a **new terminal**.

Source ROS:

```bash
source /opt/ros/jazzy/setup.bash
```

Spawn robot:

```bash
ros2 run ros_gz_sim create \
-file camera_robot.urdf \
-name camera_bot
```

This will:

* Load the URDF
* Spawn robot in Gazebo
* Start the camera sensor

You should see a **small cube robot with a camera mounted on top**.

---

# 5. Check Camera Topics

List topics:

```bash
ros2 topic list
```

Expected output:

```
/camera/image_raw
/camera/camera_info
/clock
```

Check camera info:

```bash
ros2 topic echo /camera/camera_info
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
