# ROS1 Robot Control Crash Course, with Flatland

## Installation and Environment Setup

Note: This guide assumes you are using an Ubuntu 20.04 operating system, and can be followed using either a computer running on this OS, WSL, or a Virtual Machine.

### ROS Installation

Follow the instructions in the official [ROS installation guide](http://wiki.ros.org/noetic/Installation/Ubuntu). It is recommended to at least install the desktop version, as some tools that will be used here are already contained within.

In this tutorial it is assumed that your ROS installation is placed in /opt/ros/, under the name noentic (which is the most recent Long Term Support distribution at the time of writing).

From this point on, in each bash terminal where ROS commands are used, the setup.bash script must be sourced (as explained in section 1.5 of the installation guide):

```
source /opt/ros/noetic/setup.bash
```

Instead of having to write this command everytime a new terminal is opened, the command can be written onto the .bashrc script, so that it is run automatically when opening a new terminal:

```
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Workspace Setup

Create your own workspace. Pick where you would like your workspace to be placed (usually in the home directory, a.k.a. _~/_) and create a folder with a name of your choice, for example _ros\_workspace_, and within it create a folder named _src_. Finally, from within the workspace folder, in this case the ros_workspace folder, run the _catkin\_make_ command. You can use the following commands in a terminal for this process, assuming you place your workspace in your home directory:

```
mkdir -p ~/ros_workspace/src
cd ~/ros_workspace/
catkin_make
```

Two new folders should have appeared in your workspace, _build/_ and _devel/_. Within the _devel/_ folder, you'll find multiple shell scripts. One of these, _devel/setup.bash_ will be used to allow you to run code from packages within your workspace. After you run _catkin\_make_, don't forget to execute the following command:

```
source devel/setup.bash
```

**Important Note**: Every time you open a new terminal to run ROS code from within your workspace, you must first source ROS's _setup.bash_ script and your workspace's own _setup.bash_. The command to source ROS's _setup.bash_ is already being automatically run, if you added it to _~/.bashrc_, as indicated in the _ROS Installation_ section. In order to avoid having to source the setup.bash of the workspace everytime a new terminal is opened, similarly to what was done for sourcing ROS's setup.bash, the source command can be added to the .bashrc file:
```
echo "source ~/ros_workspace/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Flatland Setup

First, clone the flatland repository into the _src_ folder of your workspace and install its dependencies (you may need to install LUA separately if you don't have it already). Then, from within your workspace, build the package with _catkin\_make_ (and source devel/setup.bash), and launch the default server to test whether everything is working. You can do this process with the following commands:

```
cd src/
git clone https://github.com/avidbots/flatland.git
cd ..
sudo apt-get update
sudo apt-get install liblua5.1-0-dev
sudo apt-get install ros-noetic-map-server
rosdep install --from-paths src --ignore-src
catkin_make
source devel/setup.bash
roslaunch flatland_server server.launch
```

If all went well, you should see a map of an office and a few models placed around, one of them moving back and forth on a cycle.

## First Run

Now that you have ROS and Flatland set up, you can start experimenting with the code from this tutorial. First clone the repository and build it:
```
cd src/
git clone https://github.com/BerserkingIdiot/flatland_quick_start_ros1.git
cd ..
catkin_make
```
You can launch the example Flatland's native visualization, or RViz, both allowing you to visualize the simulation environment and the models.
Flatland's native visualization adapts to simulation environments automatically, and provides some tools to move, spawn and delete models, and pausing the simulation. To launch this version you can use:
```
roslaunch flatland_quick_start_ros1 flatland_fviz.launch
```
RViz is more complex and requires manual setup for each new environment, but is also more flexible, in this case providing a visualization of the data generated by the LiDAR. To launch this version you can use:
```
roslaunch flatland_quick_start_ros1 flatland_rviz.launch
```
Both of these will launch a Flatland simulation, containing a small differential drive robot equipped with a LiDAR, and a few obstacles. Along with it, _rqt\_robot\_steering_, a tool that allows you to control the robot's linear and angular velocity, is also launched. Both visualization tools can be manipulated with the mouse, allowing you to drag, rotate and zoom in.

If all went well, you should see something like this (this is the RViz version, however Flatland's native visualization should look similar):

![Screenshot from 2022-04-20 10-23-27](https://user-images.githubusercontent.com/38168315/164197987-29a0a7ce-2651-4777-bcbb-07eb2cf1137f.png)

You can zoom in the visualization window with the scroll wheel and move it around by using alt + left click and dragging. The front of the robot starts pointed towards the right and the robot looks like this:

![detailed_robot](https://user-images.githubusercontent.com/38168315/175785428-7792a770-7a78-4777-91b3-50555e26e98a.png)

## Write your own code

The simulation in this package provides a small differential drive robot equipped with a LiDAR. The differential drive is controlled through Twist messages, and the LiDAR provides data in the form of LaserScan messages. In the _src/_ folder of this package, you will find some sample code that you can use to help develop your own robot controller.

Note: the LiDAR scan data is provided in the form of an array of ranges, each value corresponding to the nearest detected obstacle by that ray, or _nan_ if nothing is found. The rays are defined counter-clockwise, starting from the rear of the robot, meaning that in the first half of the array are the values for obstacles on the right and in the second half are values for obstacles on the left, with the middle of the array corresponding to the front of the robot. Playing around with the RViz visualization may help understanding how these work.

### C++

Within the _src/_ folder of this package (not to be confused with the workspace's _src/_), you can find a file named _custom\_robot\_controller.cpp_. The path to this .cpp file is _/ros\_workspace/src/flatland\_quick\_start\_ros1/src/custom\_robot\_controller.cpp_.

You can use this code to write your own code, and experiment with robot control. Don't forget to build the package after you modify the code, and to run it you must first make sure to have a simulation running. You can use one of the launch files from the previous section, for example, and then use the following command:
```
rosrun flatland_quick_start_ros1 custom_robot_controller
```

### Python

Within the _src/_ folder of this package (not to be confused with the workspace's _src/_), you can find a file named _custom_robot_controller.py_. The path to this .py file is _/ros\_workspace/src/flatland\_quick\_start\_ros1/src/custom\_robot\_controller.py_.

You can use this code to write your own code, and experiment with robot control. Unlike the C++ version, you don't have to build the package every time you modify the code, and to run it you must first make sure to have a simulation running. You can use one of the launch files from the previous section, for example, and then use the following command:
```
rosrun flatland_quick_start_ros1 custom_robot_controller.py
```

## Next Steps

### Topics, Publishers and Subscribers

In this tutorial, you were provided with a ROS node with the communication features already set up, and you might have noticed elements like the LiDAR scan subscriber and the Twist message publisher. These are essential elements of the ROS environment, and you can learn more about them in the [official ROS tutorials](http://wiki.ros.org/ROS/Tutorials).

### Exploring Flatland

In this package, there are currently 2 different maps to experiment with, found in the _flatland\_worlds/_ folder. The launch files default to the office test world, but you can use the world_path parameter to choose the world you wish to boot up, for example to boot up the maze world you can run this:
```
roslaunch flatland_quick_start_ros1 flatland_rviz.launch world_path:="maze" 
```

To learn more about the Flatland simulator, read the [documentation and tutorials](https://flatland-simulator.readthedocs.io/en/latest/)

## Feedback

Once you have finished following the tutorial, please fill out the following form: https://forms.gle/Pw1brvNvAmvSGhQr9

