# Lab: Using a Camera Sensor in Gazebo with ROS 2

## Objective

In this lab, students will learn how to:

* Launch a Gazebo simulation world
* Spawn a robot with a camera sensor using URDF
* Inspect ROS 2 topics published by the camera
* Visualize the camera stream using RViz2

---

# 1. Launch Gazebo with a World

First download the **Gazebo house world file**.

Then launch Gazebo with ROS support:

```bash
gazebo house.world -s libgazebo_ros_factory.so
```

**Note**

* The first launch may take some time because Gazebo will download the required models.
* `libgazebo_ros_factory.so` allows ROS 2 to **spawn robots dynamically** into Gazebo.

After launching, a house environment should appear in Gazebo.

---

# 2. Create the Robot URDF

Create a file named:

```
camera_robot.urdf
```

Paste the following URDF inside it.

```xml
<?xml version="1.0"?>

<robot name="camera_robot">

  <link name="base_link">
    <inertial>
      <mass value="1.0"/>
      <origin xyz="0 0 0"/>
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

  <link name="camera_link">
    <visual>
      <geometry>
        <box size="0.05 0.05 0.05"/>
      </geometry>
    </visual>
  </link>

  <joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.1 0 0.05"/>
  </joint>

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

### What this URDF does

The robot contains:

* **Base Link** – the body of the robot
* **Camera Link** – where the camera sensor is mounted
* **Fixed Joint** – attaches the camera to the robot
* **Gazebo Camera Plugin** – publishes camera images to ROS 2 topics

Camera properties:

* Resolution: **640 × 480**
* Frame rate: **30 Hz**
* Field of View: **1.396 rad (~80°)**

---

# 3. Spawn the Robot in Gazebo

Open a new terminal and run:

```bash
ros2 run gazebo_ros spawn_entity.py \
-entity camera_bot \
-file camera_robot.urdf
```

This command:

* Reads the URDF
* Spawns the robot into the Gazebo world
* Initializes the camera sensor

You should now see a **small box robot with a camera** inside the Gazebo environment.

---

# 4. Check ROS 2 Topics

Next, check what topics the camera publishes.

```bash
ros2 topic list
```

You should see topics similar to:

```
/camera/image_raw
/camera/camera_info
/clock
```

To inspect the camera stream:

```bash
ros2 topic echo /camera/camera_info
```

---

# 5. Visualize Camera in RViz2

Open RViz:

```bash
rviz2
```

Inside RViz:

1. Click **Add**
2. Select **Image**
3. Set the topic to:

```
/camera/image_raw
```

You should now see the **live camera feed from Gazebo**.


---

# Expected Outcome

Students should be able to:

* Launch Gazebo with ROS 2 integration
* Spawn a URDF robot
* Publish camera data
* Visualize camera images in RViz2

---
