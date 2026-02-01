# Building a movable robot model

**Goal:** Learn how to define movable joints in URDF, and how to add collision and inertial properties to links, and how to add joint dynamics to joints.

**Tutorial level:** Intermediate


Contents

- [The Head](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#the-head)
    
- [The Gripper](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#the-gripper)
    
- [The Gripper Arm](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#the-gripper-arm)
    
- [Other Types of Joints](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#other-types-of-joints)
    
- [Specifying the Pose](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#specifying-the-pose)
    
- [Next steps](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#next-steps)
    

In this tutorial, we’re going to revise the R2D2 model we made in the [previous tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Visual-Robot-Model-with-URDF-from-Scratch.html) so that it has movable joints. In the previous model, all of the joints were fixed. Now we’ll explore three other important types of joints: continuous, revolute and prismatic.

Make sure you have installed all prerequisites before continuing. See the [previous tutorial](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Visual-Robot-Model-with-URDF-from-Scratch.html) for information on what is required.

Again, all of the robot models mentioned in this tutorial can be found in the [urdf_tutorial](https://index.ros.org/p/urdf_tutorial) package.

[Here is the new urdf](https://github.com/ros/urdf_tutorial/blob/ros2/urdf/06-flexible.urdf) with flexible joints. You can compare it to the previous version to see everything that has changed, but we’re just going to focus on three example joints.

To visualize and control this model, run the same command as the last tutorial:
``` bash
ros2 launch urdf_tutorial display.launch.py model:=urdf/06-flexible.urdf
```

However now this will also pop up a GUI that allows you to control the values of all the non-fixed joints. Play with the model some and see how it moves. Then, we can take a look at how we accomplished this.

[![Screenshot of Flexible Model](https://raw.githubusercontent.com/ros/urdf_tutorial/ros2/images/flexible.png)](https://raw.githubusercontent.com/ros/urdf_tutorial/ros2/images/flexible.png)

## [The Head](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#id1)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Building-a-Movable-Robot-Model-with-URDF.html#the-head "Link to this heading")

``` xml
<joint name="head_swivel" type="continuous">
  <parent link="base_link"/>
  <child link="head"/>
  <axis xyz="0 0 1"/>
  <origin xyz="0 0 0.3"/>
</joint>
```

The connection between the body and the head is a continuous joint, meaning that it can take on any angle from negative infinity to positive infinity. The wheels are also modeled like this, so that they can roll in both directions forever.

The only additional information we have to add is the axis of rotation, here specified by an xyz triplet, which specifies a vector around which the head will rotate. Since we want it to go around the z axis, we specify the vector “0 0 1”.

## The Gripper
``` xml
<joint name="left_gripper_joint" type="revolute">
  <axis xyz="0 0 1"/>
  <limit effort="1000.0" lower="0.0" upper="0.548" velocity="0.5"/>
  <origin rpy="0 0 0" xyz="0.2 0.01 0"/>
  <parent link="gripper_pole"/>
  <child link="left_gripper"/>
</joint>
``` 
Both the right and the left gripper joints are modeled as revolute joints. This means that they rotate in the same way that the continuous joints do, but they have strict limits. Hence, we must include the limit tag specifying the upper and lower limits of the joint (in radians). We also must specify a maximum velocity and effort for this joint but the actual values don’t matter for our purposes here.

## The Gripper Arm
``` xml
<joint name="gripper_extension" type="prismatic">
  <parent link="base_link"/>
  <child link="gripper_pole"/>
  <limit effort="1000.0" lower="-0.38" upper="0" velocity="0.5"/>
  <origin rpy="0 0 0" xyz="0.19 0 0.2"/>
</joint>
```

The gripper arm is a different kind of joint, namely a prismatic joint. This means that it moves along an axis, not around it. This translational movement is what allows our robot model to extend and retract its gripper arm.

The limits of the prismatic arm are specified in the same way as a revolute joint, except that the units are meters, not radians.

## Other Types of Joints
There are two other kinds of joints that move around in space. Whereas the prismatic joint can only move along one dimension, a planar joint can move around in a plane, or two dimensions. Furthermore, a floating joint is unconstrained, and can move around in any of the three dimensions. These joints cannot be specified by just one number, and therefore aren’t included in this tutorial.

## Specifying the Pose

As you move the sliders around in the GUI, the model moves in Rviz. How is this done? First the [GUI](https://index.ros.org/p/joint_state_publisher_gui) parses the URDF and finds all the non-fixed joints and their limits. Then, it uses the values of the sliders to publish [sensor_msgs/msg/JointState](https://github.com/ros2/common_interfaces/blob/eloquent/sensor_msgs/msg/JointState.msg) messages. Those are then used by [robot_state_publisher](https://index.ros.org/p/robot_state_publisher) to calculate all of transforms between the different parts. The resulting transform tree is then used to display all of the shapes in Rviz.

## Adding physical and collision properties[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#adding-physical-and-collision-properties "Link to this heading")


- [Collision](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#collision)
    
- [Physical Properties](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#physical-properties)
    
    - [Inertia](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#inertia)
        
    - [Contact Coefficients](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#contact-coefficients)
        
    - [Joint Dynamics](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#joint-dynamics)
        
- [Other Tags](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#other-tags)
    
- [Next Steps](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#next-steps)
    

In this tutorial, we’ll look at how to add some basic physical properties to your URDF model and how to specify its collision properties.

### [Collision](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#id1)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#collision "Link to this heading")

So far, we’ve only specified our links with a single sub-element, `visual`, which defines (not surprisingly) what the robot looks like. However, in order to get collision detection to work or to simulate the robot, we need to define a `collision` element as well. [Here is the new urdf](https://raw.githubusercontent.com/ros/urdf_tutorial/master/urdf/07-physics.urdf) with collision and physical properties.

Here is the code for our new base link.
``` xml
<link name="base_link">
    <visual>
      <geometry>
        <cylinder length="0.6" radius="0.2"/>
      </geometry>
      <material name="blue">
        <color rgba="0 0 .8 1"/>
      </material>
    </visual>
    <collision>
      <geometry>
        <cylinder length="0.6" radius="0.2"/>
      </geometry>
    </collision>
  </link>
```
- The collision element is a direct subelement of the link object, at the same level as the visual tag.
    
- The collision element defines its shape the same way the visual element does, with a geometry tag. The format for the geometry tag is exactly the same here as with the visual.
    
- You can also specify an origin in the same way as a subelement of the collision tag (as with the visual).
    

In many cases, you’ll want the collision geometry and origin to be exactly the same as the visual geometry and origin. However, there are two main cases where you wouldn’t:

> - **Quicker Processing** Doing collision detection for two meshes is a lot more computational complex than for two simple geometries. Hence, you may want to replace the meshes with simpler geometries in the collision element.
>     
> - **Safe Zones** You may want to restrict movement close to sensitive equipment. For instance, if we didn’t want anything to collide with R2D2’s head, we might define the collision geometry to be a cylinder encasing his head to prevent anything from getting too close to his head.
>     

### [Physical Properties](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#id2)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#physical-properties "Link to this heading")

In order to get your model to simulate properly, you need to define several physical properties of your robot, i.e. the properties that a physics engine like Gazebo would need.

#### [Inertia](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#id3)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#inertia "Link to this heading")

Every link element being simulated needs an inertial tag. Here is a simple one.
``` xml
<link name="base_link">
  <visual>
    <geometry>
      <cylinder length="0.6" radius="0.2"/>
    </geometry>
    <material name="blue">
      <color rgba="0 0 .8 1"/>
    </material>
  </visual>
  <collision>
    <geometry>
      <cylinder length="0.6" radius="0.2"/>
    </geometry>
  </collision>
  <inertial>
    <mass value="10"/>
    <inertia ixx="1e-3" ixy="0.0" ixz="0.0" iyy="1e-3" iyz="0.0" izz="1e-3"/>
  </inertial>
</link>
```
- This element is also a subelement of the link object.
    
- The mass is defined in kilograms.
    
- The 3x3 rotational inertia matrix is specified with the inertia element. Since this is symmetrical, it can be represented by only 6 elements, as such.
    
    > |   |   |   |
    > |---|---|---|
    > |**ixx**|**ixy**|**ixz**|
    > |ixy|**iyy**|**iyz**|
    > |ixz|iyz|**izz**|
    
- This information can be provided to you by modeling programs such as MeshLab. The inertia of geometric primitives (cylinder, box, sphere) can be computed using Wikipedia’s [list of moment of inertia tensors](https://en.wikipedia.org/wiki/List_of_moments_of_inertia#List_of_3D_inertia_tensors) (and is used in the above example).
    
- The inertia tensor depends on both the mass and the distribution of mass of the object. A good first approximation is to assume equal distribution of mass in the volume of the object and compute the inertia tensor based on the object’s shape, as outlined above.
    
- If unsure what to put, a matrix with ixx/iyy/izz=1e-3 or smaller is often a reasonable default for a mid-sized link (it corresponds to a box of 0.1 m side length with a mass of 0.6 kg). The identity matrix is a particularly bad choice, since it is often much too high. (it corresponds to a box of 0.1 m side length with a mass of 600 kg!)
    
- You can also specify an origin tag to specify the center of gravity and the inertial reference frame (relative to the link’s reference frame).
    
- When using realtime controllers, inertia elements of zero (or almost zero) can cause the robot model to collapse without warning, and all links will appear with their origins coinciding with the world origin.
    

#### [Contact Coefficients](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#id4)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#contact-coefficients "Link to this heading")

You can also define how the links behave when they are in contact with one another. This is done with a subelement of the collision tag called contact_coefficients. There are three attributes to specify:

> - mu - [Friction coefficient](https://simple.wikipedia.org/wiki/Coefficient_of_friction)
>     
> - kp - [Stiffness coefficient](https://en.wikipedia.org/wiki/Stiffness)
>     
> - kd - [Dampening coefficient](https://en.wikipedia.org/wiki/Damping_ratio#Damping_ratio_definition)
>     

#### [Joint Dynamics](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#id5)[](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Adding-Physical-and-Collision-Properties-to-a-URDF-Model.html#joint-dynamics "Link to this heading")

How the joint moves is defined by the dynamics tag for the joint. There are two attributes here:

> - `friction` - The physical static friction. For prismatic joints, the units are Newtons. For revolving joints, the units are Newton meters.
>     
> - `damping` - The physical damping value. For prismatic joints, the units are Newton seconds per meter. For revolving joints, Newton meter seconds per radian.
>     

If not specified, these coefficients default to zero.



