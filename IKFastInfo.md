# IKFast Information
IKFast is a tool that automatically generates fast, analytical closed-form inverse kinematics solvers for a given robot configuration. It takes in a collada file (which could previously be converted from URDF in ROS1, yet that package has not yet been updated for ROS2‒this is something I am planning on doing, as changing IKFast to use URDF would be difficult) and generates a C++ file for that specific  kinematic chain. It uses nullspace exploration to find all posible solutions, which is much faster than traditional numerical methods. 

## Limitations
Currently, IKFast is only available as a Python2 file, ikfast.py, in the OpenRave library. It depends upon that library, which is currently only usable in ROS1; ROS2 documentation recommends using a Docker image of ROS1 to generate the C++ file to use as a plugin in more modern ROS2 movement planning libraries. 

IKFast also has inherent issues that it itself addresses as needing to be fixed:
>1. currently ikfast does not handle big decimal numbers well. for example defining the axes or anchors as 1.032513241 will produce very big fractions and make things slow.
>
>2. there are cases when axes align and there are infinite solutions. although ikfast can detect such cases, we need a lot more work in this area.
>
>3. for 6D ik, there are still mechanisms it cannot solve, please send the kinematics model if such a situation is encountered.
>
>4. there are 10 different types of IK, currently ray4d IK needs a lot of work.


## Solution Outline
*Input file format:* for this issue, you can either port the ROS1 package collada_urdf to ROS2 (in the original repository, this seems to have been very limitedly started) or convert ikfast.py to use URDF to begin with. 

*Python2 and OpenRave dependency:* The ikfast.py file will need to be ported to python3, and will likely need to be made to run as a standalone program. This is a large undertaking, as the file is ~10,000 lines long, which makes both porting between python versions and removing dependencies challenging and labor-intensive, though from what I've seen the dependency doesn't run too deep. The file itself claims, "For advanced users, it is also possible to use run ikfast.py as a stand-alone program, which makes it mostly independent of the OpenRAVE run-time," yet gives no instructions on how to do this. The dependencies should be removed first to simplify the python porting for automated programs. 

