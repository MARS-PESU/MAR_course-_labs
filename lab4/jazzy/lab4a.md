# Setup
please run the below commands to quickly setup anything required for the lab.
``` bash
mkdir dev_ws
cd dev_ws
mkdir src
cd src
git clone https://github.com/joshnewans/my_bot.git
cd ..
colcon build --symlink-install
```
Mobile Robots in ROS​
Since our robot is a mobile robot we'll be following some of the conventions set out in in the ROS REPs (essentially the standards):

The main coordinate frame for the robot will be called base_link (REP 105 - Coordinate Frames for Mobile Platforms)
The orientation of this coordinate frame will be X-forward, Y-left, Z-up (REP 103 - Standard Units of Measure and Coordinate Conventions)

## Creating our URDF [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#creating-our-urdf "Direct link to Creating our URDF")

Now it's time to create a URDF for our robot!

### Splitting up our files [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#splitting-up-our-files "Direct link to Splitting up our files")

For this robot, instead of keeping all our configuration in a single URDF file, we’ll be splitting it up into multiple files and _including_ them in a main file. In this post we'll be focusing on a file that contains most of the core structure of the robot, and later on we will create more files for our sensors etc. and we can include them all in here to keep things organised.

To begin with, open up `robot.urdf.xacro` from the template and delete the `base_link` that is currently there. Replace it with the line `<xacro:include filename="robot_core.xacro" />`, so that your file looks like this:

robot.urdf.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro"  name="robot">
3

4    <xacro:include filename="robot_core.xacro" />
5

6</robot>
```

If we try to relaunch `robot_state_publisher` now, it will fail, because it is trying to include a file called `robot_core.xacro` that doesn’t exist - so we better make it!

### Creating the visual structure [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#creating-the-visual-structure "Direct link to Creating the visual structure")

We're about to start creating the visual structure for our robot. Fair warning, we'll start to get into a bit of maths and geometry and spatial stuff here, which can be a bit confusing if you're not used to it. Try to follow along, but if you get confused you should be able to copy-and-paste and muddle your way to the end, and hopefully things will make more sense in hindsight.

#### Making the Core File [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#making-the-core-file "Direct link to Making the Core File")

In the `description` directory, create a new file called `robot_core.xacro`. To ensure it’s a valid URDF, we’ll start by copying the XML declaration and the `robot` tags (these are the same as in the previous file but without the `name` parameter). All of our links and joints will be going inside the robot tag, and we’ll step through them one by one.

robot\_core.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
3

4    ... all our links and joints will go in here ...
5

6</robot>
```

#### Colours [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#colours "Direct link to Colours")

As mentioned in the URDF tutorial, we have the option to declare some materials (colours) up-front so that we can use them later. The colours are specified as float RGB triplets with an alpha channel. If that's meaningless to you, don't worry too much. You can start with these colours and if you want more, use [this colour picker](https://antongerdelan.net/colour/) and copy the values from the "RGB 0.0-1.0 float" section.

You can either put these into a separate file (e.g. `colours.xacro`, remember to add your robot tags and include the new file), or at the top of this file just inside the robot tag.

robot\_core.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
3

4    <material name="white">
5        <color rgba="1 1 1 1"/>
6    </material>
7

8    <material name="orange">
9        <color rgba="1 0.3 0.1 1"/>
10    </material>
11

12    <material name="blue">
13        <color rgba="0.2 0.2 1 1"/>
14    </material>
15

16    <material name="black">
17        <color rgba="0 0 0 1"/>
18    </material>
19

20</robot>
```

#### Base Link [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#base-link "Direct link to Base Link")

As mentioned earlier, it is standard in ROS for the main "origin" link in a mobile robot to be called `base_link`. You might think the natural place for the base link is in the centre of the chassis somewhere - and this wouldn't necessarily be wrong - but for a differential drive robot it is simplest to treat the centre of the two drive wheels as the origin, since the rotation will be centred around this point. The rest of the robot can then be described from there. So we start with an empty link called `base_link`.

![](https://articulatedrobotics.xyz/assets/images/base_link-dcfef64618af852d430bc02c0c163a7c.png)

robot\_core.xacro

Expand

```xml
⋯
18    </material>
19

20    <link name="base_link">
21    </link>
22

23</robot>
```

If you want you can relaunch robot state publisher now and check everything runs (make sure to rebuild the workspace since we added a file).

#### Chassis [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#chassis "Direct link to Chassis")

Next up is our chassis. Remember, we're not so interested in the final design right now, just the rough structure, so let's make it a box that is 300 × 300 × 150mm (for our American friends, this is about 1 × 1 × 1/2 ft). URDF values are in metres, so that's 0.3 × 0.3 × 0.15m.

![](https://articulatedrobotics.xyz/assets/images/chassis_dimensions-0b3ab8a7fd4ad0fe908f8ffe96f39b0c.png)

I want the origin (the reference point) of the chassis to be at the bottom-rear-centre. So that means this `chassis` link will be connected to our `base_link` via a `fixed` joint, set back a little from the centre.

![](https://articulatedrobotics.xyz/assets/images/chassis_setback-5bb4c4173db9581f532ebd6fc35d995c.png)

One other thing to note here is that by default the box geometry will be centred around the link origin. Recall that we want the link origin at the rear-bottom of the box, so to achieve that we want to shift the box forward (in X) by half its length (0.15m), and up (in Z) by half its height (0.075m).

![](https://articulatedrobotics.xyz/assets/images/half_box_offset-de6de19c63c42c5001a5781c1c3d6381.png)

robot\_core.xacro

Expand

```xml
⋯
20    <link name="base_link">
21    </link>
22

23    <!-- CHASSIS -->
24

25    <joint name="chassis_joint" type="fixed">
26        <parent link="base_link"/>
27        <child link="chassis"/>
28        <origin xyz="-0.1 0 0"/>
29    </joint>
30

31    <link name="chassis">
32        <visual>
33            <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
34            <geometry>
35                <box size="0.3 0.3 0.15"/>
36            </geometry>
37            <material name="white"/>
38        </visual>
39    </link>
40

41</robot>
```

At this point we can relaunch `rsp.launch.py` and then start RViz in another terminal (`rviz2` or `ros2 run rviz2 rviz2`) to see the transforms and visuals displayed - it should look like the image below.

- Set the fixed frame to `base_link`
- Add a TF display (and enable showing names)
- Add a RobotModel display (setting the topic to `/robot_description`)

![](https://articulatedrobotics.xyz/assets/images/rviz_chassis-0719828b8fc781dd2a84495ede0e9d28.png)

tip

If you’d like to save your RViz setup for future use, you can do so in the file menu. Using "Save Config" will override the default, or you can use "Save Config As" to save to a new location. Then to use it, when running RViz you can use the `-d` argument and the path to the `.rviz` file. E.g. `rviz2 -d ~/path/to/my/config/file.rviz`.

note

If we wanted to, we could skip the chassis step and just add our boxes to the base link, but there are two reasons to do it this way:

- `robot_state_publisher` will issue a warning if the root link has an inertia specified.
- If we have future things physically attached to the chassis (e.g. camera, lidar), by attaching them to a `chassis` link we can move the wheels forwards or backwards if required and only change one number instead of all of them.

For our URDF to work properly we also need to add collision and inertia information, but we’re going to get all the visuals sorted out first, and then come back through and add the other stuff.

#### Drive wheels [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#drive-wheels "Direct link to Drive wheels")

Now we want to add the drive wheels. The wheels can move, so these will be connected to `base_link` via `continuous` joints. We could connect them to the chassis instead, but since we chose the base link to be at the centre of rotation it makes sense for the wheel links to be connected directly to it.

We want our wheels to be be cylinders oriented along the Y axis (left-to-right). In ROS though, cylinders by default are oriented along the Z axis (up and down). To fix this, we need to "roll" the cylinder by a quarter-turn around the X axis. I like to keep the Z-axis pointing outward (not inward), so I will rotate the left wheel clockwise (negative) around X by a quarter-turn (−π2-\\frac{\\pi}{2}−2π​ radians), and the right wheel anticlockwise (+π2+\\frac{\\pi}{2}+2π​ radians).

![](https://articulatedrobotics.xyz/assets/images/wheel_rotation-a5847977c2a062e3cd8495acaae8b95a.png)

The last item of interest is the rotation `axis`. Since our left wheel has Z facing out, a "forward" drive would be an anticlockwise (positive) rotation around the Z axis. So we want the left wheel's axis to be +1 in Z.

The full blocks for both wheels (including some chosen values for the cylinder length, radius, and Y offset - how wide the wheels are spaced apart from the centre) are shown below.

robot\_core.xacro

Expand

```xml
⋯
38        </visual>
39    </link>
40

41    <!-- LEFT WHEEL -->
42

43    <joint name="left_wheel_joint" type="continuous">
44        <parent link="base_link"/>
45        <child link="left_wheel"/>
46        <origin xyz="0 0.175 0" rpy="-${pi/2} 0 0"/>
47        <axis xyz="0 0 1"/>
48    </joint>
49

50    <link name="left_wheel">
51        <visual>
52            <geometry>
53                <cylinder length="0.04" radius="0.05" />
54            </geometry>
55            <material name="blue"/>
56        </visual>
57    </link>
58

59    <!-- RIGHT WHEEL -->
60

61    <joint name="right_wheel_joint" type="continuous">
62        <parent link="base_link"/>
63        <child link="right_wheel"/>
64        <origin xyz="0 -0.175 0" rpy="${pi/2} 0 0"/>
65        <axis xyz="0 0 -1"/>
66    </joint>
67

68    <link name="right_wheel">
69        <visual>
70            <geometry>
71                <cylinder length="0.04" radius="0.05" />
72            </geometry>
73            <material name="blue"/>
74        </visual>
75    </link>
76

77</robot>
```

If we try to view this in RViz now we'll notice that the wheels aren't displayed correctly, since nothing is publishing their joint states. We can temporarily run `joint_state_publisher_gui` to resolve this and see the wheels visualised.

```text
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```

![](https://articulatedrobotics.xyz/assets/images/jsp-ff1357cc93c6e99f1d28e97d8ae47f67.png)

#### Caster wheel [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#caster-wheel "Direct link to Caster wheel")

Finally we need to add a caster wheel/s. The easiest way to do this is to simply add a frictionless sphere, connected to our chassis with the lowest point matching the base of the wheels. This isn't the most realistic physics simulation but is simpler to set up. Depending on the design, we sometimes want multiple caster wheels, but one will do for now. We'll cover the friction side of things in the next post.

robot\_core.xacro

Expand

```xml
⋯
74        </visual>
75    </link>
76

77    <!-- CASTER WHEEL -->
78

79    <joint name="caster_wheel_joint" type="fixed">
80        <parent link="chassis"/>
81        <child link="caster_wheel"/>
82        <origin xyz="0.24 0 0" rpy="0 0 0"/>
83    </joint>
84

85    <link name="caster_wheel">
86        <visual>
87            <geometry>
88                <sphere radius="0.05" />
89            </geometry>
90            <material name="black"/>
91        </visual>
92    </link>
93

94</robot>
```

![](https://articulatedrobotics.xyz/assets/images/caster_wheel2-c168f3ec8975daedfb9e5e52a91b9c77.png)

### Adding Collision and Inertia [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#adding-collision-and-inertia "Direct link to Adding Collision and Inertia")

#### Adding collision [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#adding-collision "Direct link to Adding collision")

Once we’ve got our core visual structure tweaked the way we want it, we need to add collision. The simplest way to do this is to copy-and-paste the `geometry` and `origin` from our `<visual>` tags into `<collision>` tags.

robot\_core.xacro

Expand

```xml
⋯
31    <link name="chassis">
32        <visual>
33            <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
34            <geometry>
35                <box size="0.3 0.3 0.15"/>
36            </geometry>
37            <material name="white"/>
38        </visual>
39        <collision>
40            <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
41            <geometry>
42                <box size="0.3 0.3 0.15"/>
43            </geometry>
44        </collision>
45    </link>
⋯
61            <material name="blue"/>
62        </visual>
63        <collision>
64            <geometry>
65                <cylinder length="0.04" radius="0.05" />
66            </geometry>
67        </collision>
68    </link>
69

⋯
84            <material name="blue"/>
85        </visual>
86        <collision>
87            <geometry>
88                <cylinder length="0.04" radius="0.05" />
89            </geometry>
90        </collision>
91    </link>
92

⋯
106            <material name="black"/>
107        </visual>
108        <collision>
109            <geometry>
110                <sphere radius="0.05" />
111            </geometry>
112        </collision>
113    </link>
114

⋯
```

So go ahead and do this for all the links. To check if the collision geometry looks correct, in the RViz RobotModel display, we can untick "Visual Enabled" and tick "Collision Enabled" to see the collision geometry (it will use the colours from the visual geometry).

![](https://articulatedrobotics.xyz/assets/images/collision_display-aa360f0369279f2fc0cd5ec2e3ab9846.png)

tip

It's worth noting that this method isn’t really ideal - if we need to change any parameters (e.g. our wheels have a larger radius) we now have even more places to change them! Instead I strongly recommend you go through and use `xacro` properties to define your structure.

#### Adding inertia [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#adding-inertia "Direct link to Adding inertia")

The last things we need to add are our `<inertia>` tags. As noted in the URDF tutorial, calculating inertia values can sometimes be tricky, and it’s often easier to use macros. You can copy the entire `inertial_macros.xacro` file from the second tab and place it in your `description/` directory. Then, near the top of your `robot_core.xacro` file (or `robot.urdf.xacro`) just under the opening `<robot>` tag, add the following line to include them.

robot\_core.xacroinertial\_macros.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
3

4    <xacro:include filename="inertial_macros.xacro" />
5

⋯
```

Once that's in, go ahead and add the relevant macro to each link. Note that you'll need to specify an `<origin>` tag when using these macros. This should match the visual/collision origin, NOT the joint origin. So if the visual/inertial didn't have an origin, just specify it as all zeros. Again we have a box for the chassis, a cylinder for the drive wheels, and a sphere for the caster.

Here are my examples:

robot\_core.xacro

Expand

```xml
⋯
45            </geometry>
46        </collision>
47        <xacro:inertial_box mass="0.5" x="0.3" y="0.3" z="0.15">
48            <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
49        </xacro:inertial_box>
50    </link>
⋯
71            </geometry>
72        </collision>
73        <xacro:inertial_cylinder mass="0.1" length="0.04" radius="0.05">
74            <origin xyz="0 0 0" rpy="0 0 0"/>
75        </xacro:inertial_cylinder>
76    </link>
77

⋯
97            </geometry>
98        </collision>
99        <xacro:inertial_cylinder mass="0.1" length="0.04" radius="0.05">
100            <origin xyz="0 0 0" rpy="0 0 0"/>
101        </xacro:inertial_cylinder>
102    </link>
103

⋯
122            </geometry>
123        </collision>
124        <xacro:inertial_sphere mass="0.1" radius="0.05">
125            <origin xyz="0 0 0" rpy="0 0 0"/>
126        </xacro:inertial_sphere>
127    </link>
128

⋯
```

## Extending on this [​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf/\#extending-on-this "Direct link to Extending on this")

We now have a basic structure laid out that we can expand upon as we work on our design. As we add new aspects (sensors etc.) we can just keep creating and including more `xacro` files! This makes for a nicely modular, flexible, structured system.

The next step is to spawn our robot in Gazebo and drive it around with the keyboard. This post is long enough as it is, so the next part is in a [separate post](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-gazebo)

Here are the key files we worked on:

robot.urdf.xacrorobot\_core.xacroinertial\_macros.xacro

Expand

```xml
1<?xml version="1.0"?>
2<robot xmlns:xacro="http://www.ros.org/wiki/xacro"  name="robot">
3

4    <xacro:include filename="robot_core.xacro" />
5

6</robot>
```
