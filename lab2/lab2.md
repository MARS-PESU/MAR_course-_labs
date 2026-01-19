# Using turtlesim, ros2, and rqt — ROS 2 Documentation: Humble  documentation

**Goal:** Install and use the turtlesim package and rqt tools to prepare for upcoming tutorials.

**Tutorial level:** Beginner

start with closing all the previous terminals
then open a new terminal and run the following:
``` bash
echo -e '\n# Auto-set ROS namespace from hostname\nexport ROS_NAMESPACE=$(hostname)\necho -e "\033[1;32m[ROS 2]\033[0m Namespace set to: \033[1;34m$ROS_NAMESPACE\033[0m"' >> ~/.bashrc
```

``` bash
echo -e '\n# Auto-set ROS namespace and domain from hostname\nexport ROS_NAMESPACE=$(hostname)\nexport ROS_DOMAIN_ID=$(( $(hostname | cksum | cut -d " " -f1) % 100 ))\necho -e "\033[1;32m[ROS 2]\033[0m Namespace: \033[1;34m$ROS_NAMESPACE\033[0m | Domain ID: \033[1;36m$ROS_DOMAIN_ID\033[0m"' >> ~/.bashrc
```
then close all terminals again,
open a new terminal and then continue


### note: Take all screenshots the way it has been take in the images added below

**Time:** 15 minutes

Contents

*   [Background](#background)
    
*   [Prerequisites](#prerequisites)
    
*   [Tasks](#tasks)
    
    *   [1 Install turtlesim](#install-turtlesim)
        
    *   [2 Start turtlesim](#start-turtlesim)
        
    *   [3 Use turtlesim](#use-turtlesim)
        
    *   [4 Install rqt](#install-rqt)
        
    *   [5 Use rqt](#use-rqt)
        
    *   [6 Remapping](#remapping)
        
    *   [7 Close turtlesim](#close-turtlesim)
        
*   [Summary](#summary)
    
*   [Next steps](#next-steps)
    
*   [Related content](#related-content)
    

[Background](#id1)
[](#background "Link to this heading")
---------------------------------------------------------

Turtlesim is a lightweight simulator for learning ROS 2. It illustrates what ROS 2 does at the most basic level to give you an idea of what you will do with a real robot or a robot simulation later on.

The ros2 tool is how the user manages, introspects, and interacts with a ROS system. It supports multiple commands that target different aspects of the system and its operation. One might use it to start a node, set a parameter, listen to a topic, and many more. The ros2 tool is part of the core ROS 2 installation.

rqt is a graphical user interface (GUI) tool for ROS 2. Everything done in rqt can be done on the command line, but rqt provides a more user-friendly way to manipulate ROS 2 elements.

This tutorial touches upon core ROS 2 concepts, like nodes, topics, and services. All of these concepts will be elaborated on in later tutorials; for now, you will simply set up the tools and get a feel for them.

[Prerequisites](#id2)
[](#prerequisites "Link to this heading")
---------------------------------------------------------------

The previous tutorial, [Configuring environment](../Configuring-ROS2-Environment.html), will show you how to set up your environment.

[Tasks](#id3)
[](#tasks "Link to this heading")
-----------------------------------------------

### [1 Install turtlesim](#id4)
[](#install-turtlesim "Link to this heading")

As always, start by sourcing your setup files in a new terminal, as described in the [previous tutorial](../Configuring-ROS2-Environment.html).

Install the turtlesim package for your ROS 2 distro:

```
sudo apt update
sudo apt install ros-humble-turtlesim

```


To check if the package is installed, run the following command, which should return a list of turtlesim’s executables:

```
ros2 pkg executables turtlesim
turtlesim draw_square
turtlesim mimic
turtlesim turtle_teleop_key
turtlesim turtlesim_node

```


### [2 Start turtlesim](#id5)
[](#start-turtlesim "Link to this heading")

To start turtlesim, enter the following command in your terminal:

```
ros2 run turtlesim turtlesim_node
[INFO] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]

```


Under the command, you will see messages from the node. There you can see the default turtle’s name and the coordinates where it spawns.

The simulator window should appear, with a random turtle in the center.

![](https://docs.ros.org/en/humble/_images/turtlesim.png)

### [3 Use turtlesim](#id6)
[](#use-turtlesim "Link to this heading")

Open a new terminal and source ROS 2 again.

Now you will run a new node to control the turtle in the first node:

```
ros2 run turtlesim turtle_teleop_key

```


At this point you should have three windows open: a terminal running `turtlesim_node`, a terminal running `turtle_teleop_key` and the turtlesim window. Arrange these windows so that you can see the turtlesim window, but also have the terminal running `turtle_teleop_key` active so that you can control the turtle in turtlesim.

Use the arrow keys on your keyboard to control the turtle. It will move around the screen, using its attached “pen” to draw the path it followed so far.

Note

Pressing an arrow key will only cause the turtle to move a short distance and then stop. This is because, realistically, you wouldn’t want a robot to continue carrying on an instruction if, for example, the operator lost the connection to the robot.

You can see the nodes, and their associated topics, services, and actions, using the `list` subcommands of the respective commands:

```
ros2 node list
ros2 topic list
ros2 service list
ros2 action list

```


You will learn more about these concepts in the coming tutorials. Since the goal of this tutorial is only to get a general overview of turtlesim, you will use rqt to call some of the turtlesim services and interact with `turtlesim_node`.

### [4 Install rqt](#id7)
[](#install-rqt "Link to this heading")

Open a new terminal to install `rqt` and its plugins:

```
sudo apt update
sudo apt install '~nros-humble-rqt*'

```


To run rqt:

### [5 Use rqt](#id8)
[](#use-rqt "Link to this heading")

When running rqt for the first time, the window will be blank. No worries; just select **Plugins** > **Services** > **Service Caller** from the menu bar at the top.

### Note

It may take some time for rqt to locate all the plugins. If you click on **Plugins** but don’t see **Services** or any other options, you should close rqt and enter the command `rqt --force-discover` in your terminal.

![](https://docs.ros.org/en/humble/_images/rqt.png)
Use the refresh button to the left of the **Service** dropdown list to ensure all the services of your turtlesim node are available.

Click on the **Service** dropdown list to see turtlesim’s services, and select the `/spawn` service.

#### 5.1 Try the spawn service[](#try-the-spawn-service "Link to this heading")

Let’s use rqt to call the `/spawn` service. You can guess from its name that `/spawn` will create another turtle in the turtlesim window.

Give the new turtle a unique name, like `turtle2`, by double-clicking between the empty single quotes in the **Expression** column. You can see that this expression corresponds to the value of **name** and is of type **string**.

Next enter some valid coordinates at which to spawn the new turtle, like `x = 1.0` and `y = 1.0`.

![](https://docs.ros.org/en/humble/_images/spawn.png)
Note

If you try to spawn a new turtle with the same name as an existing turtle, like the default `turtle1`, you will get an error message in the terminal running `turtlesim_node`:

```
[ERROR] [turtlesim]: A turtle named [turtle1] already exists

```


To spawn `turtle2`, you then need to call the service by clicking the **Call** button on the upper right side of the rqt window.

If the service call was successful, you should see a new turtle (again with a random design) spawn at the coordinates you input for **x** and **y**.

If you refresh the service list in rqt, you will also see that now there are services related to the new turtle, `/turtle2/...`, in addition to `/turtle1/...`.

#### 5.2 Try the set\_pen service[](#try-the-set-pen-service "Link to this heading")

Now let’s give `turtle1` a unique pen using the `/set_pen` service:

![](https://docs.ros.org/en/humble/_images/set_pen.png)
The values for **r**, **g** and **b**, which are between 0 and 255, set the color of the pen `turtle1` draws with, and **width** sets the thickness of the line.

To have `turtle1` draw with a distinct red line, change the value of **r** to 255, and the value of **width** to 5. Don’t forget to call the service after updating the values.

If you return to the terminal where `turtle_teleop_key` is running and press the arrow keys, you will see `turtle1`’s pen has changed.

![../../../_images/new_pen.png](https://docs.ros.org/en/humble/_images/new_pen.png)

You’ve probably also noticed that there’s no way to move `turtle2`. That’s because there is no teleop node for `turtle2`.

### [6 Remapping](#id9)
[](#remapping "Link to this heading")

You need a second teleop node in order to control `turtle2`. However, if you try to run the same command as before, you will notice that this one also controls `turtle1`. The way to change this behavior is by remapping the `cmd_vel` topic.

In a new terminal, source ROS 2, and run:

```
ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel

```


Now, you can move `turtle2` when this terminal is active, and `turtle1` when the other terminal running `turtle_teleop_key` is active.

![../../../_images/remap.png](https://docs.ros.org/en/humble/_images/remap.png)

### [7 Close turtlesim](#id10)
[](#close-turtlesim "Link to this heading")

To stop the simulation, you can enter `Ctrl + C` in the `turtlesim_node` terminal, and `q` in the `turtle_teleop_key` terminals.

[Summary](#id11)
[](#summary "Link to this heading")
----------------------------------------------------

Using turtlesim and rqt is a great way to learn the core concepts of ROS 2.

[Next steps](#id12)
[](#next-steps "Link to this heading")
----------------------------------------------------------

Now that you have turtlesim and rqt up and running, and an idea of how they work, let’s dive into the first core ROS 2 concept with the next tutorial, [Understanding nodes](../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html).

[Related content](#id13)
[](#related-content "Link to this heading")
--------------------------------------------------------------------

The turtlesim package can be found in the [ros\_tutorials](https://github.com/ros/ros_tutorials/tree/humble/turtlesim) repo.

[This community contributed video](https://youtu.be/xwT7XWflMdc) demonstrates many of the items covered in this tutorial.
