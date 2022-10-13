# TIAGO-DOCKER-CUDA

The Dockerfile was modified to meed the requirements of the project [ros_contact_graspnet](https://github.com/jucamohedano/ros_contact_graspnet).

ROS Version: **Kinetic**

### Requirements

This container is meant to run with a capable GPU. It has been tested with NVIDIA RTX 2060 and NVIDIA RTX 3060.

Docker.
Docker-compose.
Nvidia docker.
CUDA and CUDNN.

### How to run the container
  
  1. To build the container run `sh install.sh`
  2. Export the following environment variables to your shell session:
    `export TIAGO_DOCKER_ROOT=<path-to-this-repository>`
    `export PATH=$PATH:$TIAGO_DOCKER_ROOT/bin`
  It is recommended that these variables are appended to your ~/.bashrc file to automate this process.
  3. Run `tiago_up` to start the container
  4. Run `tiago_bash` to execute a shell session within the container.
  3. Run `tiago_stop` to stop the container.
  
### Hints

The development workspace of the container is within the `tiago_home` directory created after the building process. 

If you want to create different development enviroments with this container it is recommended that you change follow the same installation process but change the name of the `environment variables` that were set earlier, and the name of the scripts under the `./bin` directory.

If you would like to use this development environment with PyCharm, then please refer to https://github.com/nozomisk/tiago-docker
