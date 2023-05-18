# gazebo_ros_2d_map_plugin
Gazebo simulator plugin to automatically generate a 2D occupancy map from the simulated world at a given certain height. 

This plugin was adapted from the [octomap plugin](https://github.com/ethz-asl/rotors_simulator/tree/master/rotors_gazebo_plugins) from ETH Zürich.

## Usage 
Check out the plugin in your `catkin_ws` and build it with `catkin_make`.
To include the plugin, add the following line in between the `<world> </world>` tags of your Gazebo world file:

```
<plugin name='gazebo_occupancy_map' filename='libgazebo_2d_map_plugin.so'>
    <map_resolution>0.1</map_resolution>            <!-- in meters, optional, default 0.1 -->
    <map_origin>0.0 0.0 0.0</map_origin>            <!-- in meters, optional, default [0.0, 0.0, 0.0] -->
    <map_size_x>10</map_size_x>                     <!-- in meters, optional, default 10 -->
    <map_size_y>10</map_size_y>                     <!-- in meters, optional, default 10 -->
    <init_robot_x>0</init_robot_x>                  <!-- x coordinate in meters, optional, default 0 -->
    <init_robot_y>0</init_robot_y>                  <!-- y coordinate in meters, optional, default 0 -->
</plugin>
```

The parameter `map_origin` stands for the center(origin) position of the Gazebo map calculated from the origin frame of the Gazebo. This origin position will be used for the `nav_msgs/OccupancyGrid`.

For example, if you set the **center position** of the square map to match **the origin frame** of the Gazebo, you should set the `map_origin` as 
```
<map_origin>0 0 0</map_origin>
```

Or if you set the **left-bottom position** of the square map to match **the origin frame** of the Gazebo, you should set the `map_origin` as 
```
<map_origin>$(map_size_x)/2 $(map_size_y)/2 0</map_origin>
<!-- change the value of the parameter with the shape of your map. -->
```
Then, 

The parameters `map_size_x` and `map_size_y` should the total lengths through x and y-axis. And the parameters `init_robot_x` and `init_robot_y` become x and y position of the robot calculated from the **left-bottom frame** of the Gazebo map.

To generate the map, call the `/gazebo_2d_map_plugin/generate_map` ros service:

```
rosservice call /gazebo_2d_map_plugin/generate_map
```

The generated map is published on the `/map2d` ros topic. 

You can use the `map_saver` node from the `map_server` package inside ros navigation to save your generated map to a .pgm and .yaml file:

```
rosrun map_server map_saver -f <mapname> /map:=/map2d
```
The last map generated with the ```/gazebo_2d_map_plugin/generate_map``` call is saved.

## Hints

* To identify the connected free space the robot would discover during mapping, the plugin performs a wavefront exploration along the occupancy grid starting from the origin of the gazebo world coordinate system. Please ensure that the corresponding cell is in the continuous free space. 
* The plugin will map all objects in the world, including the robot. Remove all unwanted  objects before creating the map. 
