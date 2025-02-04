
These instructions were created by Ali Tousi (stousi@missouri.edu) and G. N. DeSouza (DeSouzaG@missouri.edu)

# ROS-Noetic-and-Gazebo-in-Ubuntu-22.04
The instructions to install ROS Noetic from source in Jammy from some websites are incomplete -- one, for ex, claims to install ros-desktop-full, but it doesn't (e.g. there is no Gazebo afterwards). I have modified these instructions to install Gazebo from binary packages from OSR and ROS from source (from Focal). 



#### Install the basic stuff
sudo apt install git gnupg wget



#### First, let's install gazebo from the OSR Foundation repository
#### Make sure that there are no gazebo or ros repositories enabled/listed  under /etc/apt/source.list.d
sudo apt update

#### Clean up any previous installation of gazebo
sudo apt-get remove '.*gazebo.*' '.*sdformat.*' '.*ignition-math.*' '.*ignition-msgs.*' '.*ignition-transport.*' '.*ignition.*'

sudo apt autoremove


#### Install the dependencies for gazebo
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable ``lsb_release -cs`` main" > /etc/apt/sources.list.d/gazebo-stable.list'

sudo wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

sudo apt-get update

sudo wget https://raw.githubusercontent.com/ignition-tooling/release-tools/master/jenkins-scripts/lib/dependencies_archive.sh -O /tmp/dependencies.sh


#### Now, remove "ogre 2.1" from the list of $ogre in /tmp/dependencies.sh
sudo vi /tmp/dependencies.sh


#### Install the dependencies
GAZEBO_MAJOR_VERSION=11  ROS_DISTRO=noetic . /tmp/dependencies.sh

sudo echo $BASE_DEPENDENCIES $GAZEBO_BASE_DEPENDENCIES | tr -d '\\' | xargs sudo apt-get -y install


#### Now install gazebo binary packages from OSR
sudo apt install gazebo




####  Gazebo is done!! Now, let's install ROS from source using a FOCAL (20.04) repository
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros-latest.list' 

sudo curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

sudo apt update 


sudo apt-get install python3-rosdep python3-rosinstall-generator python3-vcstools python3-vcstool build-essential

sudo rosdep init
####  Do NOT do rosdep update yet!!!

sudo mkdir ~/Downloads

cd ~/Downloads

sudo wget http://archive.ubuntu.com/ubuntu/pool/universe/h/hddtemp/hddtemp_0.3-beta15-53_amd64.deb

sudo apt install ./hddtemp_0.3-beta15-53_amd64.deb

sudo wget https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml

sudo vi base.yaml  #######    to look like this...



###### hddtemp:
######  arch: [hddtemp]  
######  debian: [hddtemp]
######  fedora: [hddtemp]
######  freebsd: [python27]
######  gentoo: [app-admin/hddtemp]
######  macports: [python27]  
######  nixos: [hddtemp]  
######  openembedded: [hddtemp@meta-oe]  
######  opensuse: [hddtemp]  
######  rhel: [hddtemp]  
######  slackware: [hddtemp]  
######  ubuntu: 
######    '*': null   
######    bionic: [hddtemp]    
######    focal: [hddtemp]    
######    impish: [hddtemp]    
######    jammy: [hddtemp]
    


####  Save it
sudo mv base.yaml /etc/ros/rosdep/sources.list.d/

sudo vi /etc/ros/rosdep/sources.list.d/20-default.list    #####   to look like this


######
#yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml

yaml file:///etc/ros/rosdep/sources.list.d/base.yaml
######


#### Download list of packages to install noetic-desktop-full
sudo rosdep update

sudo mkdir ~/ros_catkin_ws

cd ~/ros_catkin_ws

sudo rosinstall_generator desktop_full --rosdistro noetic --deps --tar > noetic-desktop-full.rosinstall

sudo mkdir ./src

sudo vcs import --input noetic-desktop-full.rosinstall ./src

sudo rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic -y

#### Backup
cd ~/ros_catkin_ws/src

sudo mkdir ~/Downloads/backup

sudo mv rosconsole urdf ~/Downloads/backup/

#### Download and use fixed branch
sudo git clone https://github.com/dreuter/rosconsole.git

cd rosconsole

sudo git checkout noetic-jammy

cd ~/ros_catkin_ws/src

sudo git clone https://github.com/dreuter/urdf.git

cd urdf

sudo git checkout set-cxx-version


#### Now, we compile it from source.  DO NOT use -j20 as it may overload your CPU
cd ~/ros_catkin_ws

sudo ./src/catkin/bin/catkin_make_isolated -j10 -l10 --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/noetic 

#### Done!
