# 安装ubuntu mate 16.04

## 下载ubuntu

从官网下载ubutnu mate 地址为 https://ubuntu-mate.org/raspberry-pi/ubuntu-mate-16.04.2-desktop-armhf-raspberry-pi.img.xz.torrent
还有其他版本的https://ubuntu-pi-flavour-maker.org/download/
## 刷入ubuntu
这个需要使用工具Win32 Disk Mangaer 下载地址 https://sourceforge.net/projects/win32diskimager/

插入读卡机，选择相应的盘符，载入之前下载好的镜像文件，点击write等待写入成功即可完成ubuntu的刷入
-------------------------------------------------------------------------------------------------------------------------
#                                    安装ros

sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
sudo apt-get update
sudo apt-get -y install ros-kinetic-ros-base ros-kinetic-slam-gmapping ros-kinetic-navigation ros-kinetic-xacro ros-kinetic-yocs-velocity-smoother ros-kinetic-robot-state-publisher ros-kinetic-joint-state-publisher ros-kinetic-teleop-twist-*
sudo rosdep init
rosdep update
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc

 
# 测试结果
安装完成，运行roscore可以看到下面输出即可标识完成ros安装

-------------------------------------------------------------------------------------------------------------------------
#                                    开启SSH
1. 打开终端
2. sudo raspi-config
3. 进入第三项
4. 选则p2回车
5. 选择yes回车
6. 等待完成回车确认
7. 退出配置，ps -e | grep ssh 看到sshd即开启
--------------------------------------------------------------------------------------------------------------------------
#                            扩展SWAP空间  先关闭swap
cd /var 
sudo swapoff /var/swap 

#重设swap大小1M*4096=4GB,会花较长时间，请耐心等待 
sudo dd if=/dev/zero of=swap bs=1M count=4096
 
#格式化 
sudo mkswap /var/swap 

#开启swap 
sudo swapon /var/swap 

#设置开机启动，在/etc/fstab文件中添加如下代码 
/var/swap swap swap defaults 0 0 
#查看当前已生效的swap 
swapon -s 
#查看当前swap使用情况 
free -m


----------------------------------------------------------------------------------------------------------------------------
#                              安装cartographer
##安装编译工具
sudo apt-get update
sudo apt-get install -y python-wstool python-rosdep ninja-build
##
mkdir catkin_ws_carto
cd catkin_ws_carto
wstool init src

wstool merge -t src https://raw.githubusercontent.com/googlecartographer/cartographer_ros/master/cartographer_ros.rosinstall
##特别说明，在执行wstool update -t src之前，需要将src/.rosinstall文件（Ctrl+h显示隐藏文件）修改成以下内容,以解决ceres-solver下载不了的问题
##           https://github.com/ceres-solver/ceres-solver.git
##
wstool update -t src
##安装依赖项
src/cartographer/scripts/install_proto3.sh
sudo rosdep init
rosdep update

rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y
##编译和安装
catkin_make_isolated --install --use-ninja
##source
source catkin_ws_carto/install_isolated/setup.bash
##运行2D  demo
roslaunch cartographer_ros demo_backpack_2d.launch bag_filename:=${HOME}/Downloads/cartographer_paper_deutsches_museum.bag
-------------------------------------------------------------------------------------------------------
#                        ros配置主从机
# 1.主机为PC      IP：192.162.0.113             NAME: sch
#   从机为树莓派  IP：192.162.0.108             NAME: raspi

# 2.从机配置（树莓派配置）
#  将主机的IP添加到主机的 /etc/hosts 这个配置文件里
 sudo gedit /etc/hosts
# 添加： 192.162.0.113  sch（也就是打开文件后一般为第三行添加，在一行注释之前）
 gedit  ~/.bashrc     
# 添加： export ROS_MASTER_URI=http://192.162.0.113:11311         此处为PC的IP
#        export ROS_HOSTNAME=192.162.0.108                       此处为树莓派IP

# 3.主机配置
#      将从机的IP添加到主机的 /etc/hosts 这个配置文件里
sudo gedit /etc/hosts 
# 添加：192.162.0.108  raspi
gedit ~/.bashrc
# 添加： export ROS_MASTER_URI=http://192.162.0.113:11311            
#        export ROS_HOSTNAME=192.162.0.108

# 4.测试：（只要主机开启roscore即可）
# 测试主从机联通：
#   在主机上，ping从机：ping 192.162.0.108
#   在从机上，ping主机：ping 192.162.0.113

# 测试ROS主从配置：
#   主机上，新终端执行：  roscore
#   从机上，新终端执行：  rostopic list
------------------------------------------------------------------------------------------------------
#               linux下非root用户获得/dev/ttyUSB×的读写权限

raspi@raspi:~$ sudo usermod -aG dialout raspi
#重启
------------------------------------------------------------------------------------------------------
#               树莓派 ~/.bashrc设置
source ~/mrobot_ws/devel/setup.bash

export ROS_MASTER_URI=http://192.168.0.113:11311 主机IP
export ROS_HOSTNAME=192.168.0.108 从机IP


