# Composing multiple nodes in a single process


## Discover available components

To see what components are registered and available in the workspace:

```
ros2 component types
```

The terminal will return the list of all available components:

```
(... components of other packages here)
composition
  composition::Talker
  composition::Listener
  composition::Server
  composition::Client
  ```
  
## Run-time composition using ROS services with a publisher and subscriber

In the first shell, start the component container:

```
ros2 run rclcpp_components component_container
```

Open the second shell and verify that the container is running via ros2 command line tools:

```
ros2 component list
```

You should see a name of the component:

```
/ComponentManager
```

In the second shell load the talker component (see talker source code):

```
ros2 component load /ComponentManager composition composition::Talker
```

The command will return the unique ID of the loaded component as well as the node name:

```
Loaded component 1 into '/ComponentManager' container node as '/talker'
```

Now the first shell should show a message that the component was loaded as well as repeated message for publishing a message.

##

Run another command in the second shell to load the listener component (see listener source code):

```
ros2 component load /ComponentManager composition composition::Listener
```

Terminal will return:

```
Loaded component 2 into '/ComponentManager' container node as '/listener'
```

The ros2 command line utility can now be used to inspect the state of the container:

```
ros2 component list
```

You will see the following result:

```
/ComponentManager
   1  /talker
   2  /listener
```

Now the first shell should show repeated output for each received message


## Run-time composition using ROS services with a server and client

In the first shell:

```
ros2 run rclcpp_components component_container
```

In the second shell (see server and client source code):

```
ros2 component load /ComponentManager composition composition::Server
ros2 component load /ComponentManager composition composition::Client
```

In this case the client sends a request to the server, the server processes the request and replies with a response, and the client prints the received response.


## Run-time composition using dlopen

This demo presents an alternative to run-time composition by creating a generic container process and explicitly passing the libraries to load without using ROS interfaces. The process will open each library and create one instance of each “rclcpp::Node” class in the library source code

```
ros2 run composition dlopen_composition `ros2 pkg prefix composition`/lib/libtalker_component.so `ros2 pkg prefix composition`/lib/liblistener_component.so
```

## Composition using launch actions

While the command line tools are useful for debugging and diagnosing component configurations, it is frequently more convenient to start a set of components at the same time. To automate this action, we can use the functionality in ros2 launch.

```
ros2 launch composition composition_demo.launch.py
```


# Advanced Topics

Now basic operation of components have been done, we can take a look for more advanced topics.

## Unloading components

In the first shell, start the component container:

```
ros2 run rclcpp_components component_container
```

Verify that the container is running via ros2 command line tools:

```
ros2 component list
```

You should see a name of the component:

```
/ComponentManager
```

In the second shell (see talker source code). The command will return the unique ID of the loaded component as well as the node name.

```
ros2 component load /ComponentManager composition composition::Talker
ros2 component load /ComponentManager composition composition::Listener
```

Use the unique ID to unload the node from the component container.

```
ros2 component unload /ComponentManager 1 2
```

The terminal should return:

```
Unloaded component 1 from '/ComponentManager' container
Unloaded component 2 from '/ComponentManager' container
```

In the first shell, verify that the repeated messages from talker and listener have stopped.



## Remapping container name and namespace

The component manager name and namespace can be remapped via standard command line arguments:

```
ros2 run rclcpp_components component_container --ros-args -r __node:=MyContainer -r __ns:=/ns
```

In a second shell, components can be loaded by using the updated container name:

```
ros2 component load /ns/MyContainer composition composition::Listener
```

## Remap component names and namespaces

Component names and namespaces may be adjusted via arguments to the load command.

In the first shell, start the component container:

```
ros2 run rclcpp_components component_container
```

Some examples of how to remap names and namespaces.

Remap node name:

```
ros2 component load /ComponentManager composition composition::Talker --node-name talker2
```

Remap namespace:

```
ros2 component load /ComponentManager composition composition::Talker --node-namespace /ns
```

Remap both:

```
ros2 component load /ComponentManager composition composition::Talker --node-name talker3 --node-namespace /ns2
```

Now use ros2 command line utility:

```
ros2 component list
```

In the console you should see corresponding entries:

```
/ComponentManager
   1  /talker2
   2  /ns/talker
   3  /ns2/talker
```

## Passing parameter values into components

The ros2 component load command-line supports passing arbitrary parameters to the node as it is constructed. This functionality can be used as follows:

```
ros2 component load /ComponentManager image_tools image_tools::Cam2Image -p burger_mode:=true
```

## Passing additional arguments into components

The ros2 component load command-line supports passing particular options to the component manager for use when constructing the node. As of now, the only command-line option that is supported is to instantiate a node using intra-process communication. This functionality can be used as follows:

```
ros2 component load /ComponentManager composition composition::Talker -e use_intra_process_comms:=true
```





