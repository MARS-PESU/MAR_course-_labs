
## Setup

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

### Mobile Robots in ROS[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#mobile-robots-in-ros "Direct link to Mobile Robots in ROS")

Since our robot is a mobile robot we'll be following some of the conventions set out in in the ROS REPs (essentially the standards):

- The main coordinate frame for the robot will be called `base_link` ([REP 105 - Coordinate Frames for Mobile Platforms](https://www.ros.org/reps/rep-0105.html#base-link))
- The orientation of this coordinate frame will be X-forward, Y-left, Z-up ([REP 103 - Standard Units of Measure and Coordinate Conventions](https://www.ros.org/reps/rep-0103.html#axis-orientation))

## Working With URDF Files[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#working-with-urdf-files "Direct link to Working With URDF Files")

Let's start by covering some basics of how to work with URDF files.

### Quick Recap[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#quick-recap "Direct link to Quick Recap")

First, we should recap how to publish a robot description using URDFs. We start off with a bunch of files in the URDF format, that together describe our robot.

These files are processed by a tool called `xacro` which combines them into a single, complete URDF. This is passed to `robot_state_publisher` which makes the data available on the `/robot_description` topic, and also broadcasts the appropriate transforms.

If there are any joints that move, `robot_state_publisher` will expect to see the input values published on the `/joint_states` topic, and while doing our initial tests we can use the `joint_state_publisher_gui` to fake those values.

![](https://articulatedrobotics.xyz/assets/images/rsp-2afa5bef6c72583be919186c08605d9f.png)

To make life easier, we typically wrap this up into a launch file.

Before we move on, we also want to make sure that `xacro` and `joint_state_publisher_gui` are installed (if they aren't already):

```
sudo apt install ros-humble-xacro ros-humble-joint-state-publisher-gui
```

### Launching and Visualising[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#launching-and-visualising "Direct link to Launching and Visualising")

If you made a copy of the package from the GitHub template repo, you should already have a launch file (`launch/rsp.launch.py`) and a `description` directory containing a very basic URDF, `robot.urdf.xacro`.

![](https://articulatedrobotics.xyz/assets/images/file_tree-d6383e16ee90537350f131568b9df87f.png)

Try launching that file with `ros2 launch my_bot rsp.launch.py` (where `my_bot` is whatever you called your package, mine is `articubot_one`). You should see the output displayed below:

![](https://articulatedrobotics.xyz/assets/images/launch_rsp-c1cb421546e6a00c5fbbecd3f9cfc2a4.png)

Because this URDF file only has a single link (and it's empty), there won't be anything to display in RViz yet. Once we have added a couple of links and something to display (at the end of the _Chassis_ section below), we'll be able to open RViz and do the following:

- Set the fixed frame to `base_link`
- Add a TF display (and enable showing names)
- Add a RobotModel display (setting the topic to `/robot_description`)

![](https://articulatedrobotics.xyz/assets/images/rviz_chassis-0719828b8fc781dd2a84495ede0e9d28.png)

> Note: If you’d like to save your RViz setup for future use, you can do so in the file menu. Then to use it, when running RViz you can use the `-d` argument and the path to the `.rviz` file. E.g. `rviz2 -d ~/path/to/my/config/file.rviz`.

### Tricks when making changes to URDFs[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#tricks-when-making-changes-to-urdfs "Direct link to Tricks when making changes to URDFs")

Before we start creating our URDF files there are a couple of things to be aware of when saving our changes.

- If you build your workspace with `colcon build --symlink-install` then you don’t need to rebuild every time you update the URDF, it will be automatic EXCEPT whenever you add a new file. That file needs to get built/copied once, and after that will be kept updated.
- You’ll need to quit and relaunch `robot_state_publisher` each time you make a change.
- RViz sometimes won’t pick up all your changes straight away. In this case you'll need to hit the “Reset” button in the bottom corner. If it’s still not updating, try ticking and unticking the display items, or closing and reopening the program.

## Creating our URDF[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#creating-our-urdf "Direct link to Creating our URDF")

Now it's time to create a URDF for our robot!

### Splitting up our files[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#splitting-up-our-files "Direct link to Splitting up our files")

For this robot, instead of keeping all our configuration in a single URDF file, we’ll be splitting it up into multiple files and _including_ them in a main file. In this post we'll be focusing on a file that contains most of the core structure of the robot, and later on we will create more files for our sensors etc. and we can include them all in here to keep things organised.

To begin with, open up `robot.urdf.xacro` from the template and delete the `base_link` that is currently there. Replace it with the line `<xacro:include filename="robot_core.xacro" />`, so that your file looks like this:

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro"  name="robot">

    <xacro:include filename="robot_core.xacro" />

</robot>
```

If we try to relaunch `robot_state_publisher` now, it will fail, because it is trying to include a file called `robot_core.xacro` that doesn’t exist - so we better make it!

### Creating the visual structure[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#creating-the-visual-structure "Direct link to Creating the visual structure")

We're about to start creating the visual structure for our robot. Fair warning, we'll start to get into a bit of maths and geometry and spatial stuff here, which can be a bit confusing if you're not used to it. Try to follow along, but if you get confused you should be able to copy-and-paste and muddle your way to the end, and hopefully things will make more sense in hindsight.

**Making the Core File**

In the `description` directory, create a new file called `robot_core.xacro`. To ensure it’s a valid URDF, we’ll start by copying the XML declaration and the `robot` tags (these are the same as in the previous file but without the `name` parameter). All of our links and joints will be going inside the robot tag, and we’ll step through them one by one.

``` xml
<?xml version="1.0"?>  
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">  
  
... all our links and joints will go in here ...  
  
</robot>
```

**Colours**

As mentioned in the URDF tutorial, we have the option to declare some materials (colours) up-front so that we can use them later. The colours are specified as float RGB triplets with an alpha channel. If that's meaningless to you, don't worry too much. You can start with these colours and if you want more, use [this colour picker](https://antongerdelan.net/colour/) and copy the values from the "RGB 0.0-1.0 float" section.

You can either put these into a separate file (e.g. `colours.xacro`, remember to add your robot tags and include the new file), or at the top of this file just inside the robot tag.

``` xml
<material name="white">  
<color rgba="1 1 1 1"/>  
</material>  
  
<material name="orange">  
<color rgba="1 0.3 0.1 1"/>  
</material>  
  
<material name="blue">  
<color rgba="0.2 0.2 1 1"/>  
</material>  
  
<material name="black">  
<color rgba="0 0 0 1"/>  
</material>
```

**Base Link**

As mentioned earlier, it is standard in ROS for the main "origin" link in a mobile robot to be called `base_link`. You might think the natural place for the base link is in the centre of the chassis somewhere - and this wouldn't necessarily be wrong - but for a differential drive robot it is simplest to treat the centre of the two drive wheels as the origin, since the rotation will be centred around this point. The rest of the robot can then be described from there. So we start with an empty link called `base_link`.

![](https://articulatedrobotics.xyz/assets/images/base_link-dcfef64618af852d430bc02c0c163a7c.png)

``` xml
<link name="base_link">
</link>
```

If you want you can relaunch robot state publisher now and check everything runs (make sure to rebuild the workspace since we added a file).

**Chassis**

Next up is our chassis. Remember, we're not so interested in the final design right now, just the rough structure, so let's make it a box that is 300 × 300 × 150mm (for our American friends, this is about 1 × 1 × 1/2 ft). URDF values are in metres, so that's 0.3 × 0.3 × 0.15m.

![](https://articulatedrobotics.xyz/assets/images/chassis_dimensions-0b3ab8a7fd4ad0fe908f8ffe96f39b0c.png)

I want the origin (the reference point) of the chassis to be at the bottom-rear-centre. So that means this `chassis` link will be connected to our `base_link` via a `fixed` joint, set back a little from the centre.

![](https://articulatedrobotics.xyz/assets/images/chassis_setback-5bb4c4173db9581f532ebd6fc35d995c.png)

One other thing to note here is that by default the box geometry will be centred around the link origin. Recall that we want the link origin at the rear-bottom of the box, so to achieve that we want to shift the box forward (in X) by half its length (0.15m), and up (in Z) by half its height (0.075m).

![](https://articulatedrobotics.xyz/assets/images/half_box_offset-de6de19c63c42c5001a5781c1c3d6381.png)

``` xml
<joint name="chassis_joint" type="fixed">
    <parent link="base_link"/>
    <child link="chassis"/>
    <origin xyz="-0.1 0 0"/>
</joint>

<link name="chassis">
    <visual>
        <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
        <geometry>
            <box size="0.3 0.3 0.15"/>
        </geometry>
        <material name="white"/>
    </visual>
</link>
```

At this point we can fire up RViz as described earlier to see the transforms and visuals displayed - it should look like the image below.

![](https://articulatedrobotics.xyz/assets/images/rviz_chassis-0719828b8fc781dd2a84495ede0e9d28.png)

> Note: If we wanted to, we could skip the chassis step and just add our boxes to the base link, but there are two reasons to do it this way:

- `robot_state_publisher` will issue a warning if the root link has an inertia specified.
- If we have future things physically attached to the chassis (e.g. camera, lidar), by attaching them to a `chassis` link we can move the wheels and only change one number instead of all of them.

For our URDF to work properly we also need to add collision and inertia information, but we’re going to get all the visuals sorted out first, and then come back through and add the other stuff.

**Drive wheels**

Now we want to add the drive wheels. The wheels can obviously move, so these will be connected to `base_link` via `continuous` joints. We could connect them to the chassis instead, but since we chose the base link to be at the centre of rotation it makes sense for the wheel links to be connected directly to it.

We want our wheels to be be cylinders oriented along the Y axis (left-to-right). In ROS though, cylinders by default are oriented along the Z axis (up and down). To fix this, we need to "roll" the cylinder by a quarter-turn around the X axis. I like to keep the Z-axis pointing outward (not inward), so I will rotate the left wheel clockwise (negative) around X by a quarter-turn (−π2−2π​ radians), and the right wheel anticlockwise (+π2+2π​ radians).

![](https://articulatedrobotics.xyz/assets/images/wheel_rotation-a5847977c2a062e3cd8495acaae8b95a.png)

The last item of interest is the rotation `axis`. Since our left wheel has Z facing out, a "forward" drive would be an anticlockwise (positive) rotation around the Z axis. So we want the left wheel's axis to be +1 in Z.

The full blocks for both wheels (including some chosen values for the cylinder length, radius, and Y offset - how wide the wheels are spaced apart from the centre) are shown below.

```xml
<!-- LEFT WHEEL -->

<joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <origin xyz="0 0.175 0" rpy="-${pi/2} 0 0"/>
    <axis xyz="0 0 1"/>
</joint>

<link name="left_wheel">
    <visual>
        <geometry>
            <cylinder length="0.04" radius="0.05" />
        </geometry>
        <material name="blue"/>
    </visual>
</link>
```

``` xml
<!-- RIGHT WHEEL -->

<joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <origin xyz="0 -0.175 0" rpy="${pi/2} 0 0"/>
    <axis xyz="0 0 -1"/>
</joint>

<link name="right_wheel">
    <visual>
        <geometry>
            <cylinder length="0.04" radius="0.05" />
        </geometry>
        <material name="blue"/>
    </visual>
</link>
```


If we try to view this in RViz now we'll notice that the wheels aren't displayed correctly, since nothing is publishing their joint states. We can temporarily run `joint_state_publisher_gui` to resolve this and see the wheels visualised.

![](https://articulatedrobotics.xyz/assets/images/jsp-ff1357cc93c6e99f1d28e97d8ae47f67.png)

**Caster wheel**

Finally we need to add a caster wheel/s. The easiest way to do this is to simply add a frictionless sphere, connected to our chassis with the lowest point matching the base of the wheels. This isn't the most realistic physics simulation but is simpler to set up. Depending on the design, we sometimes want multiple caster wheels, but one will do for now. We'll cover the friction side of things in the next post.

``` xml
<!-- CASTER WHEEL -->

<joint name="caster_wheel_joint" type="fixed">
    <parent link="chassis"/>
    <child link="caster_wheel"/>
    <origin xyz="0.24 0 0" rpy="0 0 0"/>
</joint>

<link name="caster_wheel">
    <visual>
        <geometry>
            <sphere radius="0.05" />
        </geometry>
        <material name="black"/>
    </visual>
</link>
```

![](https://articulatedrobotics.xyz/assets/images/caster_wheel2-c168f3ec8975daedfb9e5e52a91b9c77.png)

### Adding Collision and Inertia[​](https://articulatedrobotics.xyz/tutorials/mobile-robot/concept-design/concept-urdf#adding-collision-and-inertia "Direct link to Adding Collision and Inertia")

**Adding collision**

Once we’ve got our core visual structure tweaked the way we want it, we need to add collision. The simplest way to do this is to copy-and-paste the `geometry` and `origin` from our `<visual>` tags into `<collision>` tags.

For example:

``` xml

<link name="chassis">
    <visual>
        <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
        <geometry>
            <box size="0.3 0.3 0.15"/>
        </geometry>
        <material name="white"/>
    </visual>
    <collision>
        <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
        <geometry>
            <box size="0.3 0.3 0.15"/>
        </geometry>
    </collision>
</link>
```

So go ahead and do this for all the links. To check if the collision geometry looks correct, in the RViz RobotModel display, we can untick "Visual Enabled" and tick "Collision Enabled" to see the collision geometry (it will use the colours from the visual geometry).

![](https://articulatedrobotics.xyz/assets/images/collision_display-aa360f0369279f2fc0cd5ec2e3ab9846.png)

It's worth noting that this method isn’t really ideal - if we need to change any parameters (e.g. our wheels have a larger radius) we now have even more places to change them! Instead I strongly recommend you go through and use `xacro` properties to define your structure.

**Adding inertia**

The last thing we need to add are our `<inertia>` tags. As noted in the URDF tutorial, calculating inertia values can sometimes be tricky, and it’s often easier to use macros. Download the `inertia_macros.xacro` file from [here](https://github.com/joshnewans/articubot_one/blob/d5aa5e9bc9039073c0d8fd7fe426e170be79c087/description/inertial_macros.xacro) and place it in your `description/` directory. Then, near the top of your `robot_core.xacro` file (or `robot.urdf.xacro`) just under the opening `<robot>` tag, add the following line to include them.

``` xml
<xacro:include filename="inertial_macros.xacro" />
```

Once that's in, go ahead and add the relevant macro to each link. Note that you'll need to specify an `<origin>` tag when using these macros. This should match the visual/collision origin, NOT the joint origin. So if the visual/inertial didn't have an origin, just specify it as all zeros.

Here are my examples:

```xml
<link name="chassis">
     <!-- ... visual and collision here ... -->
    <xacro:inertial_box mass="0.5" x="0.3" y="0.3" z="0.15">
        <origin xyz="0.15 0 0.075" rpy="0 0 0"/>
    </xacro:inertial_box>
</link>

<link name="left_wheel"> 
    <!-- ... visual and collision here ... -->
    <xacro:inertial_cylinder mass="0.1" length="0.05" radius="0.05">
        <origin xyz="0 0 0" rpy="0 0 0"/>
    </xacro:inertial_cylinder>
</link>

<!-- Right wheel is same as left for inertia -->

<link name="caster_wheel">
    <!-- ... visual and collision here ... -->
    <xacro:inertial_sphere mass="0.1" radius="0.05">
        <origin xyz="0 0 0" rpy="0 0 0"/>
    </xacro:inertial_sphere>
</link>
```
