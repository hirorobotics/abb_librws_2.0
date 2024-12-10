# abb_librws2.0
<!-- 
[![Build Status: Ubuntu Bionic (Actions)](https://github.com/ros-industrial/abb_librws/workflows/CI%20-%20Ubuntu%20Bionic/badge.svg?branch=master)](https://github.com/ros-industrial/abb_librws/actions?query=workflow%3A%22CI+-+Ubuntu+Bionic%22)
[![Build Status: Ubuntu Focal (Actions)](https://github.com/ros-industrial/abb_librws/workflows/CI%20-%20Ubuntu%20Focal/badge.svg?branch=master)](https://github.com/ros-industrial/abb_librws/actions?query=workflow%3A%22CI+-+Ubuntu+Focal%22)
[![Github Issues](https://img.shields.io/github/issues/ros-industrial/abb_librws.svg)](http://github.com/ros-industrial/abb_librws/issues) -->

[![license - bsd 3 clause](https://img.shields.io/:license-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

<!-- [![support level: community](https://img.shields.io/badge/support%20level-community-lightgray.svg)](http://rosindustrial.org/news/2016/10/7/better-supporting-a-growing-ros-industrial-software-platform) -->

## Important Notes

> **Note:** This fork addresses the incompatibilities with the new Omnicore controller and RWS 2.0 starting from the [original repo](https://github.com/ros-industrial/abb_librws). The repo will not be merged since it introduces breaking changes with the upstream.

RobotWare versions `7.0` and higher are currently compatible with this version of *abb_librws*. On the other hand, RobotWare versions strictly lower than `7.0` are not compatible (due to RWS `1.0` being replaced by RWS `2.0`).

See [this](https://developercenter.robotstudio.com/api/rwsApi/) for RWS 1.0 documentation and [this](https://developercenter.robotstudio.com/api/RWS) for RWS 2.0 documentation.

ROS users may use any of the following build tools to build the library:

* ROS 1: `catkin_make_isolated` or [catkin_tools](https://catkin-tools.readthedocs.io/en/latest/index.html).
* ROS 2: [colcon](https://colcon.readthedocs.io/en/released/).

## Overview

This repo is a C++ library for interfacing with ABB robot controllers supporting *Robot Web Services* (RWS) `2.0`. 
exploited to control the ABB robots with ROS2.

Please note that this package has not been productized, it is provided "as-is" and only limited support can be.

### Requirements
* RobotWare version `7.0` or **higher**.

### Dependencies

* [POCO C++ Libraries](https://pocoproject.org) (`>= 1.4.3` due to WebSocket support). Install it with:

``` bash 
sudo apt-get install libpoco-dev
```

### Limitations

The original version of [abb_librws](https://github.com/ros-industrial/abb_librws) provided several features that we are slowly porting in the newer version. Please, [create an issue](https://github.com/hirorobotics/abb_librws_2.0/issues) if something does not work as intended or create a [pull request](https://github.com/hirorobotics/abb_librws_2.0/pulls) if you have fixed/added one:

* [ ] Reading/writing of IO-signals.
* [ ] Reading/writing of RAPID data.
* [ ] Reading of RAPID data properties.
* [ ] Starting/stopping/resetting the RAPID program.
* [ ] Subscriptions (i.e. receiving notifications when resources are updated).
* [ ] Uploading/downloading/removing files.
* [ ] Checking controller state (e.g. motors on/off, auto/manual mode and RAPID execution running/stopped).
* [x] Reading the joint/Cartesian values of a mechanical unit.
* [ ] Register as a local/remote user (e.g. for interaction during manual mode).
* [ ] Turning the motors on/off.
* [x] Reading of current RobotWare version and available tasks in the robot system.

## Important concepts

This section aims to provide a basic understanding of how this repo and RWS (Robot Web Services) in general works with ABB robots.

### Code Stack
This repo is part of a bigger stack composed of:
- [abb_libegm](https://github.com/ros-industrial/abb_libegm).It offers the possibility to interact with ABB robots through their real time interface called EGM.
- abb_egm_rws_managers. It is a wrapper of the current repo and abb_libegm to ease their usage. 
- [abb_omnicore_ros2](https://github.com/hirorobotics/abb_omnicore_ros2). It is a fork from [abb_ros2](https://github.com/PickNikRobotics/abb_ros2) that supports ABB IRC5 controllers only. It exploits abb_egm_rws_managers and implements a [ros2_control hardware interface](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html).
- abb_ros2_msgs, which are custom ROS2 messages for the ABB controllers.
- StateMachine Add-In ([1.0](https://robotapps.robotstudio.com/#/viewApp/7fa7065f-457f-47ce-98d7-c04882e703ee) or [1.1](https://robotapps.robotstudio.com/#/viewApp/c163de01-792e-4892-a290-37dbe050b6e1)). An optional *RobotWare Add-In* that can be useful when configuring an ABB robot controller for use with these libraries. See more info at [its section](#statemachine-add-in-optional).

On the other hand, this repo can work also by itself, without the introduced stack.

### How RWS communicates with the ABB controller 

The following is a conceptual sketch of how this RWS library can be viewed, in relation to an ABB robot controller as well as the EGM companion library mentioned above. The optional *StateMachine Add-In* is related to the robot controller's RAPID program and system configuration.

![RWS sketch](docs/images/rws_sketch.png)

## File content and explanation

The following is a tree-like structure of the main files and their content:

```
abb_librws_2.0/
├── include/
│   └── abb_librws/
│       ├── rws_client.h                # Inherits from `POCOClient` and provides interaction methods for using the RWS services and resources.
│       ├── rws_interface.h             # Encapsulates an `RWSClient` instance and provides more user-friendly methods for using the RWS services and resources.
│       ├── rws_poco_client.h           # Sets up and manages HTTP and WebSocket communication and is unaware of the RWS protocol.
│       └── rws_state_machine_interface.h # Inherits from `RWSInterface` and has been designed to interact with the aforementioned *StateMachine Add-In*. The interface knows about the custom RAPID variables and routines, as well as system configurations, loaded by the RobotWare Add-In.
```

The optional *StateMachine Add-In* for RobotWare can be used in combination with any of the classes above, but it works especially well with the `RWSStateMachineInterface` class.

### StateMachine Add-In [Optional]

> **Note:** This part has not yet been tested with the current repo. Feedback on this are more than welcome.

The purpose of the RobotWare Add-In is to *ease the setup* of ABB robot controllers. It is made for both *real controllers* and *virtual controllers* (simulated in RobotStudio). If the Add-In is selected during a RobotWare system installation, then the Add-In will load several RAPID modules and system configurations based on the system specifications (e.g. number of robots and present options).

The RAPID modules and configurations constitute a customizable, but ready to run, RAPID program which contains a state machine implementation. Each motion task in the robot system receives its own state machine instance, and the intention is to use this in combination with external systems that require interaction with the robot(s). The following is a conceptual sketch of the RAPID program's execution flow.

<p align="center">
  <img src="docs/images/statemachine_addin_sketch.png" width="500">
</p>

To install the Add-In:

1. Go to the *Add-Ins* tab in RobotStudio.
2. Search for *StateMachine Add-In* in the *RobotApps* window.
3. Select the desired Add-In version and retrieve it by pressing the *Add* button.
4. Verify that the Add-In was added to the list *Installed Packages*.
5. The Add-In should appear as an option during the installation of a RobotWare system.

See the Add-In's user manual ([1.0](https://robotapps.blob.core.windows.net/appreferences/docs/27e5bd15-b5ec-401d-986a-30c9d2934e97UserManual.pdf) or [1.1](https://robotapps.blob.core.windows.net/appreferences/docs/cd504500-80e2-4cb6-9419-c60ea4ad6d56UserManual.pdf)) for more details, as well as for install instructions for RobotWare systems. The manual can also be accessed by right-clicking on the Add-In in the *Installed Packages* list and selecting *Documentation*.

## Acknowledgements

The **core development** has been supported by the European Union's Horizon 2020 project [SYMBIO-TIC](http://www.symbio-tic.eu/).
The SYMBIO-TIC project has received funding from the European Union's Horizon 2020 research and innovation programme under grant agreement no. 637107.

<img src="docs/images/symbio_tic_logo.png" width="250">

The **open-source process** has been supported by the European Union's Horizon 2020 project [ROSIN](http://rosin-project.eu/).
The ROSIN project has received funding from the European Union's Horizon 2020 research and innovation programme under grant agreement no. 732287.

<img src="docs/images/rosin_logo.png" width="250">

The opinions expressed reflects only the author's view and reflects in no way the European Commission's opinions.
The European Commission is not responsible for any use that may be made of the contained information.

Currently, this repo is mantained by the [Hiro Robotics srl](https://www.hirorobotics.com/):

<img src="docs/images/Hiro-Robotics-Logo.png" width="250">

### Special Thanks

Special thanks to [gavanderhoorn](https://github.com/gavanderhoorn) for guidance with open-source practices and ROS-Industrial conventions.
