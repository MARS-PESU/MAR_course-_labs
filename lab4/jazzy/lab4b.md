# Gazebo Simulation

Continuing our concept design, in this post we'll take our URDF from the previous tutorial, drop it into the Gazebo simulator, and drive it around!

## Spawning our robot in Gazebo [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#spawning-our-robot-in-gazebo "Direct link to Spawning our robot in Gazebo")

Now that we have the rough shape of our robot worked out, it’s time to get it up and running in the Gazebo simulator.

Let's start by spawning our robot into Gazebo as-is, and we'll recap the important parts of the [Gazebo overview tutorial](https://articulatedrobotics.xyz/tutorials/ready-for-ros/gazebo) while we're at it.

### Launch robot\_state\_publisher with sim time [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#launch-robot_state_publisher-with-sim-time "Direct link to Launch robot_state_publisher with sim time")

When we are running nodes with a Gazebo simulation, it's good practice to always set the `use_sim_time` parameter to `true`, which ensures that all the parts of the system agree on how to count time and can synchronise properly. This includes `robot_state_publisher`, so whether we run it directly (with `ros2 run`) or with our launch file (`rsp.launch.py`) we should make sure we set that parameter.

Launch `robot_state_publisher` with sim time using the following command (substituting your package name for `my_bot`):

```bash
ros2 launch my_bot rsp.launch.py use_sim_time:=true
```

Now it should be running and publishing the full URDF to `/robot_description`.

### Launch Gazebo with ROS compatibility [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#launch-gazebo-with-ros-compatibility "Direct link to Launch Gazebo with ROS compatibility")

tip

If you haven't already installed Gazebo, you can do so using `sudo apt install ros-jazzy-ros-gz`.

Next up we need to run Gazebo, using the launch file provided by the `ros_gz_sim` package.

```bash
ros2 launch ros_gz_sim gz_sim.launch.py gz_args:=empty.sdf
```

This should open an "empty" Gazebo window. Make sure you tell it to use the built-in `empty.sdf` world which is a big flat plane - if you leave it actually empty there will be no ground and your robot will fall down!

### Spawning our robot [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#spawning-our-robot "Direct link to Spawning our robot")

Finally, we can spawn our robot using the `create` program provided by `ros_gz_sim`. Run the following command to do this (the entity name here doesn't really matter, you can put whatever you like). I also like to spawn the robot a little in the air (positive Z axis) to make sure it doesn't clip through the floor.

```bash
ros2 run ros_gz_sim create -topic robot_description -name my_bot -z 0.1
```

We should now see our robot appear in the Gazebo window. We can't drive it just yet, but that's ok, we'll fix that soon.

![](https://articulatedrobotics.xyz/assets/images/spawned-gazebo-new-417a1b33e8f5c07a9e254c45419c0b4a.png)

## Simulation improvements [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#simulation-improvements "Direct link to Simulation improvements")

### Creating a launch file [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#creating-a-launch-file "Direct link to Creating a launch file")

Our first improvement will be to create a launch file for running the simulation, to avoid having to close and rerun all three of these programs every time we make a change.

Create a new file in your `launch/` directory called `launch_sim.launch.py` and paste the contents of the code block below.

warning

Make sure you change the package name (line 21) to whatever you have called your package!

launch/launch\_sim.launch.py

Expand

```python
1import os
2

3from ament_index_python.packages import get_package_share_directory
4

5

6from launch import LaunchDescription
7from launch.actions import IncludeLaunchDescription, DeclareLaunchArgument
8from launch.launch_description_sources import PythonLaunchDescriptionSource
9from launch.substitutions import LaunchConfiguration
10

11from launch_ros.actions import Node
12

13

14

15def generate_launch_description():
16

17

18    # Include the robot_state_publisher launch file, provided by our own package. Force sim time to be enabled
19    # !!! MAKE SURE YOU SET THE PACKAGE NAME CORRECTLY !!!
20

21    package_name='my_bot' #<--- CHANGE ME
22

23    rsp = IncludeLaunchDescription(
24                PythonLaunchDescriptionSource([os.path.join(\
25                    get_package_share_directory(package_name),'launch','rsp.launch.py'\
26                )]), launch_arguments={'use_sim_time': 'true'}.items()
27    )
28
29    world = LaunchConfiguration('world')
30

31    world_arg = DeclareLaunchArgument(
32        'world',
33        default_value="empty.sdf",
34        description='World to load'
35        )
36

37    # Include the Gazebo launch file, provided by the ros_gz_sim package
38    gazebo = IncludeLaunchDescription(
39                PythonLaunchDescriptionSource([os.path.join(\
40                    get_package_share_directory('ros_gz_sim'), 'launch', 'gz_sim.launch.py')]),
41                    launch_arguments={'gz_args': ['-r -v4 ', world], 'on_exit_shutdown': 'true'}.items()
42             )
43

44    # Run the spawner node from the ros_gz_sim package. The entity name doesn't really matter if you only have a single robot.
45    spawn_entity = Node(package='ros_gz_sim', executable='create',
46                        arguments=['-topic', 'robot_description',\
47                                   '-name', 'my_bot',\
48                                   '-z', '0.1'],
49                        output='screen')
50

51

52

53    # Launch them all!
54    return LaunchDescription([\
55        rsp,\
56        world_arg,\
57        gazebo,\
58        spawn_entity,\
59    ])
```

Take a minute to read through the file and get a general understanding of what it does (you don't need to understand every word right now). In this file we:

- "Include" the `rsp.launch.py` file, from our package, and force `use_sim_time` to be true
- Declare a launch argument `world` so that we can change the simulated world
- "Include" the Gazebo launch file (`gz_sim.launch.py`), from the `ros_gz_sim` package with various arguments:
  - `-r` starts playback
  - `-v4` increases logging
  - `world` is our launch argument above
  - `on_exit_shutdown` tells it to quit everything if Gazebo stops
- Run the `create` node from `ros_gz_sim`

After rebuilding and making sure all the previous programs have closed (including `rsp.launch.py`), we can try running this launch file. We should see our robot in Gazebo, exactly like before, only now we have an easier way get there.

```text
ros2 launch my_bot launch_sim.launch.py
```

### Note on `gazebo` tags [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#note-on-gazebo-tags "Direct link to note-on-gazebo-tags")

We saw in the Gazebo introduction tutorial that we can improve our Gazebo simulation by adding `<gazebo>` tags to our URDF file. Previous versions of Gazebo required us to add a `<material>` to every link for colours to work, but now it is able to use the colour from the URDF, so we only need `<gazebo>` tags when simulation-specific information must be added.

```xml
<link name="my_link">
    <!-- All the stuff that is inside the link tag -->
</link>

<gazebo reference="my_link">
    <!-- Gazebo-specific parameters for that link -->
</gazebo>

<gazebo>
    <!-- Gazebo parameters that are not specific to a link -->
</gazebo>
```

tip

Some people like to create a whole extra `xacro` file for this to keep the simulation-specific stuff away from the core robot, but I like to keep them together so that it’s more obvious to me if I’ve written something contradictory (e.g. made the RViz and Gazebo colours different). It’s up to you what you want to do.

### Friction [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#friction "Direct link to Friction")

One important aspect of a good simulation is setting the friction parameters appropriately. For this particular robot you could skip this completely, or you could dive deep into tuning every detail of the simulation, but it's worth making at least this change.

We want to remove the friction on the caster wheel. We have modelled it as a fixed sphere but it is typically a smooth rolling surface, or very low-friction. Adding the following block below our `caster_wheel` link will disable friction in both directions, preventing it from dragging on the ground.

description/robot\_core.xacro

Expand

```xml
⋯
128        </xacro:inertial_sphere>
129    </link>
130

131    <gazebo reference="caster_wheel">
132        <mu1 value="0.0"/>
133        <mu2 value="0.0"/>
134    </gazebo>
135

136</robot>
```

tip

Something that always confused me was that the SDF spec mentions `mu` and `mu2` as the friction settings (under some `ode` parameters), while lots of examples have `mu1` and `mu2`. This is because the [conversion from URDF to SDF](http://sdformat.org/tutorials?tut=sdformat_urdf_extensions) expects the latter and converts it to the former. Speaking of friction in Gazebo, [here](https://www.allisonthackston.com/articles/ignition-vs-gazebo.html#friction) is an interesting analysis which shows why it can be so confusing.

### Torque & Velocity [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#torque--velocity "Direct link to Torque & Velocity")

Another change we will make is to limit the torque and velocity of our drive wheels. I've chosen `0.05 N.m` for the effort (torque) and `10.0 rad/s` for the angular velocity. You may choose to set your values higher (or lower), but without these, the simulation will respond instantly to changes which can feel a little "jerky". Note these changes are not inside a `<gazebo>` tag as the `<limit>` tag is part of the URDF spec.

description/robot\_core.xacro

Expand

```xml
⋯
54    <joint name="left_wheel_joint" type="continuous">
55        <parent link="base_link"/>
56        <child link="left_wheel"/>
57        <origin xyz="0 0.175 0" rpy="-${pi/2} 0 0"/>
58        <axis xyz="0 0 1"/>
59        <limit effort="0.05" velocity="10.0"/>
60    </joint>
⋯
81    <joint name="right_wheel_joint" type="continuous">
82        <parent link="base_link"/>
83        <child link="right_wheel"/>
84        <origin xyz="0 -0.175 0" rpy="${pi/2} 0 0"/>
85        <axis xyz="0 0 -1"/>
86        <limit effort="0.05" velocity="10.0"/>
87    </joint>
⋯
```

### Spherical Collision [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#spherical-collision "Direct link to Spherical Collision")

One last change to make is using a _sphere_ as the collision geometry for our drive wheels. We can keep the radius the same as the cylinder we previously had, but now we have an infinitely small point of contact with the ground and will minimise weird odometry issues caused by the friction of the differential motion. (To see this in action, run Gazebo and RViz without this change and do a full rotation on the spot. While Gazebo gets back to its initial position, the RViz representation will be out-of-sync).

In our URDF we need to update the `collision` geometry (not visual!) for both the left and right links by replacing `cylinder` with `sphere` and removing the `length` parameter. The screenshot below was from the old Gazebo where you could more easily see the collision geometry, but you can do the same by toggling the collision geometry in the `RobotModel` in RViz.

description/robot\_core.xacro

Expand

```xml
⋯
62    <link name="left_wheel">
63        <visual>
64            <geometry>
65                <cylinder length="0.04" radius="0.05" />
66            </geometry>
67            <material name="blue"/>
68        </visual>
69        <collision>
70            <geometry>
71                <sphere radius="0.05" />
72            </geometry>
73        </collision>
⋯
89    <link name="right_wheel">
90        <visual>
91            <geometry>
92                <cylinder length="0.04" radius="0.05" />
93            </geometry>
94            <material name="blue"/>
95        </visual>
96        <collision>
97            <geometry>
98                <sphere radius="0.05" />
99            </geometry>
⋯
```

![](https://articulatedrobotics.xyz/assets/images/sphere_collision-5a20a2a33a818653c4ec18d1d2a3ce24.png)

note

If you have a better solution for this, I'd love to know!

Soon we'll actually drive our simulated robot to see if we got these right!

## Controlling Gazebo via ROS [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#controlling-gazebo-via-ros "Direct link to Controlling Gazebo via ROS")

### Understanding control in Gazebo [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#understanding-control-in-gazebo "Direct link to Understanding control in Gazebo")

Before we start doing the work to get our robot driving, there are a couple of concepts we should take a moment to cover.

The first thing to note is that later on in the project we'll be using the fantastic `ros2_control` library to handle our control code.
What's cool about that is the same code will work for both the simulated robot AND the real robot, which minimises the differences between the two.

Since `ros2_control` is a bit complicated to set up, and the concepts surrounding it are worth spending time understanding well, for now we'll use a more simple differential-drive control system that comes with Gazebo, and just take a brief look at the concepts.

When we have a real robot, it will have a control system. The main thing that control system will do is take a command velocity input (how fast we want the robot to be going), translate that into motor commands for the motor drivers, read the actual motor speeds back out, and calculate the true velocity.

With ROS, that command velocity is on a topic called `/cmd_vel`, and the classic type is `Twist`, which is just six numbers - linear velocity in the x y and z axes, and angular velocity around each axis. For a differential drive robot though, we can only control two things: linear speed in x (driving forward and backwards), and angular speed in z (turning), so the other four numbers will always be 0. Newer systems like we will be using will usually use `TwistStamped` which adds the time stamp and reference frame.

info

Some of these diagrams require updating for newer versions but the overall principles are unchanged.

![](https://articulatedrobotics.xyz/assets/images/control-real-00de4dc8bbcf4a3de96169b6bb1298f9.png)

For some applications, like mapping, we are more interested in the robot position than its velocity. The control system can estimate this for us by integrating the velocity over time, adding it up in tiny little time steps. This process is called dead reckoning, and the resulting position estimate is called our odometry.

In the Gazebo overview tutorial, we saw that whenever we want to use ROS to interact with Gazebo, we do it with plugins. The control system will be a plugin (`ros2_control` or, for now, `gz::sim::systems::DiffDrive`) and that will interact with the core Gazebo code which simulates the motors and physics.

![](https://articulatedrobotics.xyz/assets/images/control-gazebo-384f04461cd23554a67f150690ce0415.png)

And this whole system then interacts with our diagram from the previous post. Instead of faking the joint states (with `joint_state_publisher_gui`), the Gazebo robot is spawned from `/robot_description`, and the joint\_states are published by plugins. The plugins also broadcast a transfrom from a new frame called `odom` (which is like the world origin, the robot's start position), to `base_link`, which lets any other code know the current position estimate for our robot.

![](https://articulatedrobotics.xyz/assets/images/gazebo-full-2764dbb5bf6ada175a4ff4b49381372d.png)

### Adding control plugins [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#adding-control-plugins "Direct link to Adding control plugins")

So to drive our robot around we'll need to add a control plugin to our URDF. Instead of putting more into our core file, we’re now going to create a new `xacro` file called `gazebo_control.xacro`, and add the include for it to our root file, `robot.urdf.xacro`.

After adding the XML declaration and robot tags, we want to add a `<gazebo>` tag, and inside that is where we’ll put our content.

description/gazebo\_control.xacrodescription/robot.urdf.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
3

4    <gazebo>
5

6        <!-- Content will go here! -->
7

8    </gazebo>
9

10</robot>
```

Inside those `<gazebo>` tags, we will create a `<plugin>` tag to load the `DiffDrive` plugin. We also need to load the `JointStatePublisher` plugin to publish the joint states.

Copy and paste the whole plugin tag from below, and take a look through the various parameters:

- Check that the `Wheel Information` section is correct (in case you used different measurements).
- The `Input` and `Output` sections tell Gazebo how to interact with the ROS topics and transforms. Leave these alone for now.

You can see all the available tags at the [DiffDrive API docs](https://gazebosim.org/api/sim/8/classgz_1_1sim_1_1systems_1_1DiffDrive.html).

gazebo\_control.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
3

4    <gazebo>
5        <plugin name="gz::sim::systems::DiffDrive" filename="gz-sim-diff-drive-system">
6

7            <!-- Wheel Information -->
8            <left_joint>left_wheel_joint</left_joint>
9            <right_joint>right_wheel_joint</right_joint>
10            <wheel_separation>0.35</wheel_separation>
11            <wheel_radius>0.05</wheel_radius>
12

13            <!-- Input -->
14            <topic>cmd_vel</topic>
15

16            <!-- Output -->
17            <frame_id>odom</frame_id>
18            <child_frame_id>base_link</child_frame_id>
19            <odom_topic>odom</odom_topic>
20            <tf_topic>tf</tf_topic>
21

22        </plugin>
23

24

25        <plugin name="gz::sim::systems::JointStatePublisher" filename="gz-sim-joint-state-publisher-system">
26            <topic>joint_states</topic>
27            <joint_name>left_wheel_joint</joint_name>
28            <joint_name>right_wheel_joint</joint_name>
29        </plugin>
30    </gazebo>
31

32

33</robot>
```

### Bridge to ROS [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#bridge-to-ros "Direct link to Bridge to ROS")

In the new Gazebo, there is a whole separate set of topics that live internal to Gazebo and we need to _bridge_ the data from there to ROS (and vice versa). We will add new file `gz_bridge.yaml` in the `config` directory which contains a list of topics that need bridging.

We specify:

- The topic name and type on the ROS side
- The topic name and type on the Gazebo side
- The direction it should map data

We also need to edit our `launch_sim.launch.py` file to launch the bridge node (called `parameter_bridge`) with our config.

config/gz\_bridge.yamllaunch/launch\_sim.launch.py

Expand

```yaml
1# Clock needed so ROS understand's Gazebo's time
2- ros_topic_name: "clock"
3  gz_topic_name: "clock"
4  ros_type_name: "rosgraph_msgs/msg/Clock"
5  gz_type_name: "gz.msgs.Clock"
6  direction: GZ_TO_ROS
7

8# Command velocity subscribed to by DiffDrive plugin
9- ros_topic_name: "cmd_vel"
10  gz_topic_name: "cmd_vel"
11  ros_type_name: "geometry_msgs/msg/TwistStamped"
12  gz_type_name: "gz.msgs.Twist"
13  direction: ROS_TO_GZ
14

15# Odometry published by DiffDrive plugin
16- ros_topic_name: "odom"
17  gz_topic_name: "odom"
18  ros_type_name: "nav_msgs/msg/Odometry"
19  gz_type_name: "gz.msgs.Odometry"
20  direction: GZ_TO_ROS
21

22# Transforms published by DiffDrive plugin
23- ros_topic_name: "tf"
24  gz_topic_name: "tf"
25  ros_type_name: "tf2_msgs/msg/TFMessage"
26  gz_type_name: "gz.msgs.Pose_V"
27  direction: GZ_TO_ROS
28

29# Joint states published by JointState plugin
30- ros_topic_name: "joint_states"
31  gz_topic_name: "joint_states"
32  ros_type_name: "sensor_msgs/msg/JointState"
33  gz_type_name: "gz.msgs.Model"
34  direction: GZ_TO_ROS
```

Take a moment to review each entry in `gz_config.yaml` and understand why it is included, as well as the launch file changes.

add the `ros_gz_bridge` to `launch/launch_sim.launch.py`
``` python
def generate_launch_description():
    # Other code

    bridge_config = os.path.join(
        get_package_share_directory(package_name), 
        'config', 
        'gz_bridge.yaml'
    )

    ros_gz_bridge = Node(
        package='ros_gz_bridge',
        executable='parameter_bridge',
        parameters=[{'config_file': bridge_config}],
        output='screen'
    )

    return LaunchDescription([
        rsp,
        world_arg,
        gazebo,
        ros_gz_bridge, # add this
        spawn_entity
    ])
```

### Testing control [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#testing-control "Direct link to Testing control")

If we relaunch Gazebo now, our robot will be sitting there, ready to accept a `TwistStamped` command velocity on the `/cmd_vel` topic. The easiest way for us to produce that is with a tool called `teleop_twist_keyboard`.

note

To break down that name: `teleop` is short for teleoperation, i.e. remote operation by a human as opposed to autonomous control. `Twist` is the type of message ROS uses to combine the linear and angular velocities of an object. And `keyboard` is because we are using the keyboard to control it. However we actually want to use a `TwistStamped` (has the time and coordinate frame associated) for better compatibility with newer nodes.

Go ahead and run it using the following command:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -p stamped:=true -p use_sim_time:=true
```

That should produce the window below, with instructions on how to use it. If we start pressing the keys (e.g. `i` to move forward) then we should start to see the robot moving around.

![](https://articulatedrobotics.xyz/assets/images/teleop-c8016223642383006aafa61131ab8684.png)

Annoying Controls

Something you may find confusing/annoying/unintuitive about this tool is that it can only respond to input while the terminal window is active. It may be easiest to shrink the window down so that you can have it visible on top of your Gazebo window, and if you accidentally click away (e.g. to move the Gazebo camera) remember to switch back to the terminal.

A far more practical approach is to use a different package: `teleop_twist_joy` (specifically the `teleop_node`) which, combined with `joy_node` (from the `joy` package) gives the operator the ability to send command velocities using a controller, even when the terminal isn't active. We'll cover this in much more detail down the track when we dive deep into the control system, but if you're feeling adventurous you may want to experiment with it now.

## Using our Simulation [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#using-our-simulation "Direct link to Using our Simulation")

### Visualising the Result [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#visualising-the-result "Direct link to Visualising the Result")

Now that we have our Gazebo plugin broadcasting a transform from `odom` to `base_link`, we should be able to see this in RViz.
Start RViz, and add the `TF` and `RobotModel` displays like in the last tutorial.

Now, set the fixed frame to `odom`. As we drive the robot around in Gazebo, we should see its motion matched in the RViz display.

![](https://articulatedrobotics.xyz/assets/images/rviz-407746cc167ca7bf884102e52f76abf0.png)

### The Problem with Plugins [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#the-problem-with-plugins "Direct link to The Problem with Plugins")

warning

I'm not sure whether the advice below is the "correct" way to deal with Gazebo plugins, but it's worth reading to avoid/understand crashes down the track!

Much of Gazebo's functionality is defined by plugins, many of which you can find listed [here](https://gazebosim.org/api/sim/8/namespacegz_1_1sim_1_1systems.html). How and when these plugins are loaded can be confusing. Here are a set of observations based on [these](https://gazebosim.org/docs/latest/sdf_worlds/) [two](https://gazebosim.org/api/sim/8/server_config.html) pages and my own testing that will hopefully help you to manage plugins well:

- As far as we are concerned, plugins can come from three different places: default config, world SDF, or robot URDF.
- There are three core plugins (`Physics`, `UserCommands`, and `SceneBroadcaster`) which are loaded by default based on the file `~/.gz/sim/8/server.config`. If you manage to load a genuinely empty world, these plugins will be loaded.
- When you launch Gazebo with a world file (SDF) to load, if that SDF has _any_ plugins, even different to the default ones, then the default ones will not be loaded.
- If you spawn a robot after loading (even very quickly like we do), any plugins defined inside `<gazebo>` tags in its URDF (which is converted at runtime to SDF) will be loaded in addition to the default or world plugins.
- Certain plugins (e.g. `Sensors`) cannot be loaded twice or it will kill Gazebo. Others (e.g. `DiffDrive`) seem to cope fine.
- I don't think there is anyway to check if a plugin already exists before loading it (maybe a good feature in future?)
- The default `empty.sdf` loads those default plugins explictly, but also the `Contact` plugin, which I guess must be important?
- Both the `filename` and `name` fields of the `<plugin>` tag must be correct (I think in the old Gazebo the `name` was arbitrary).

Based on this, the approach taken in these tutorials is to ensure that at least following plugins are loaded in the SDF file:

```xml
<plugin filename="gz-sim-physics-system" name="gz::sim::systems::Physics"/>
<plugin filename="gz-sim-user-commands-system" name="gz::sim::systems::UserCommands"/>
<plugin filename="gz-sim-scene-broadcaster-system" name="gz::sim::systems::SceneBroadcaster"/>
<plugin filename="gz-sim-sensors-system" name="gz::sim::systems::Sensors"/>
<plugin filename="gz-sim-contact-system" name="gz::sim::systems::Contact"/>
```

Additional plugins can be loaded by the robot/s via their URDF when they spawn.

### Making an Obstacle Course [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#making-an-obstacle-course "Direct link to Making an Obstacle Course")

Constructing a Gazebo world is outside the scope of this tutorial, but in short you want to:

- Add objects to your environment using the Resource Spawner
- Delete your robot using the Entity Tree so that it doesn't get included
- Use `Menu -> Save world as...` to save the file as a `.sdf`
- Edit that SDF to make sure the above plugin tags are all included

To load it back up we can supply the world path to our launch file. I store my worlds in a `world` directory so that when I launch from the workspace root it looks something like:

```bash
ros2 launch my_bot launch_sim.launch.py world:=src/my_bot/worlds/my_cool_world.sdf
```

See if you can make an obstacle course for your robot to drive around in!

![](https://articulatedrobotics.xyz/assets/images/obstacle2-bb118b0ab7be40a034c85848d3f17c16.png)

info

I really like the `Construction Barrel`, `Construction Cone`, and `Jersey Barrier` for making test worlds, but these appear washed-out in the new Gazebo. There is a way to fix this per-computer by modifying `~/.gz/fuel/fuel.gazebosim.org/openrobotics/models/construction cone/3/meshes/construction_cone.dae` but it is a bit annoying. If anyone knows how to raise this with the appropriate author that would be appreciated!

### Dealing with problems [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo/\#dealing-with-problems "Direct link to Dealing with problems")

Beyond the general [Gazebo issues](https://articulatedrobotics.xyz/tutorials/ready-for-ros/gazebo#under-the-hood--dealing-with-crashes), there are a couple of things to check if the robot isn't driving quite right:

- Use `ros2 topic list`, `ros2 topic echo /cmd_vel`, and `ros2 topic info /cmd_vel` (or other topic names) to check the data is there and has sensible speeds etc.
- Use the `Topic Viewer` and `Topic Echo` built-in to Gazebo (three dots in the top right) to check the internal simulation data is correct
- Make sure your inertia values are sensible, especially the masses
- Play around with the friction settings
- Check that the plugin parameters all make sense, especially the wheel separation and diameter
