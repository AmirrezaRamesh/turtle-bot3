# turtle-bot3
tutorial of setting turtlebot3 up and also running some tests on it

### what is turtle bot?



# section 1

## setting up the turtlebot3 waffle pi
in this document, we're trying to set up hardware, firmware and software of turtlebot3 waffle pi

## list
- [important note](#important-note)
- [overveiw](#overveiw)
- [steps](#steps)
- [troubleshooting list](#trouble-shooting-list)
  
  
## important-note
all the nessccery commands, packages and examples of turtlebot is avaibles on e-manual web page 
if you want to start using turtle bot make sure you follow the instrucstion of e-manual humble tutorial step by step

link : https://emanual.robotis.com/docs/en/platform/turtlebot3 .

this github README is only to document the extra details & correction that we encountred along the way, not a full document.
thank you!

## overveiw 
- robot: turtlebot3
- model: waffle pi
- SBC: raspberry pi 3 (16 GB micro SD, 1 GB RAM)
- control board : OpenCR 1.0
- Lidar : LDS-1.0
- network : local enviroment
- server : ubuntu server 22.04 LTS
- client : ubuntu desktop 22.04 LTS
- ROS distro : ROS2 Humble
- assemble status : already assmembled 


## steps
let's reveiw the steps that I did accourding to e-manual : 
- burn the image of ubuntu server on microSD using Pi Imager (check TB 1) 
- put the microSD in raspbery pi and turn on the power of OpenCR which is connected to RPi
- let the ubuntu server boot and login using `ubuntu` as both username and password
- cd to `/etc/netplan/` direcotory and modify the .yaml file using `vim` to connect to internet
- use `ip a` command and check the IP assigned to `wlan0`
- use the command `ssh ubuntu@192.168.X.X` from remote PC to connect to RPi(check TB2)
- now move that to e-manual and install all the required dependecies of ROS & turtlebot & opencr(checkout TB3)
- add the required scripts and commands to /.bashrc including `export TURTLEBOT3_MODEL= waffle_pi` and etc
- reboot the RPi and ssh again
- run the command `ros2 launch turtlebot3_bringup robot.launch.py` for robot bring up
- now test the `ros2 launch turtlebot3_teleop teleop_keyboard` and see if you can move the robot using keyboard
- if all the above steps are done successfully, robot is ready to be used 


## trouble-shooting-list

### TB1
on the e-manual web page, above the screen you can see the `noetic` & `humble` options as desirable ROS distros and we should choose the `humble` distro. on the `sbc setup` section, at the beginning it says to download a image and locally give it to Pi Imager but that is not correct and the script is not updated. this Image is for `noetic` version and we should install the ubuntu server 22.04 Image which can be choosed from the gui of the imager. the first option is `choose OS`. click on the option and naviagte to `other GPOS` and choose `ubuntu server 22.04.5 LTS 65-bit` and start the writing. 

### TB2
after trying to ssh to raspberry pi, we got the error `premission denied(publickey)` and since we wanted to ssh using password not public key, I ran `sudo vim /etc/ssh/sshd_config` and uncommented the `passwordauthentiocation yes` and also changed the `yes` of `publickeyauthentication yes` to `no` so it would only use the password authentication. also uncommented the `permitrootlogin prohibit-password` and change it to `permitrootlogin yes` then restared the ssh service using `sudo systemctl restart ssh*`. but then again we got the error `premission denied()`. I should mention that we could ssh from raspberry to remote PC but not the other way around. my geuss is that its related to some security option of ubuntu server/ so the method using public key that worked for me is noted here: 
- after making sure `openssh-server` and `openssh-client` is installed on remote PC, run the command `ssh-keygen -t rsa` on it to create publickey
- then on ubuntu server, run the command `sudo scp UserofRemotePC@IPofRemotePC:/home/user/.ssh/id_rsa.pub /root/` and now file is moved on /root/ directory
- on ubuntu server use the command `cat /root/id_rsa.pub > /home/ubuntu/.ssh/authorized_keys` and also `cat /root/id_rsa.pub > /root/.ssh/authorized_keys`
- now restart service using `sudo systemctl restart ssh*`
 after these command we can remotely ssh to ubuntu server without need of any password by just knowing the local IP of ubuntu server
important note: along the way of using these commands, maybe ubuntu server locks the ssh connection if the footprint of remotePC is not recognized by it. in that case you need to use the command `sudo ssh-keygen -R 192.168.X.X` on ubuntu server remember that the IP in the command is the local IP of remote PC

### TB3
at first there was a 8GB microSD card on RPi. after downloading turtle-bot packages there was a `$ colcon build --symlink-install --parallel-workers 1` command to build the workspace. after running this command, it took 40 minutes and then it froze on 28% of process. tried it multiple times and still the output was the same. it froze the 28%. when I searched it on internet, other people had the same problem with raspberry pi 3. the problem was due to lack of RAM which is only 1GB on RPi3. the solution was **to add 2GB to 4GB of swap**. when I tried to do so, it seemed the there was no available space on microSD card since it was only 8GB and the packages of ubutu, ROS and turtle but had already made up all the space. so we paurchased a new SDcard with 16GB of space. then I used the commands below to make a 4GB command and then the `colcon build` command was done successfully after about an hour:
- `sudo swapoff /swapfile #if already on`
- `sudo fallocate -l 4G /swapfile`
- `sudo chmod 600 /swapfile`
- `sudo mkswap /swapfile`
- `sudo swapon /swapfile`
- `sudo vim /etc/fstab` and add the line `/swapfile swap swap defaults 0 0` at the end of script
the swap is correctly set up. use `htop` and see if it's shown below `memory` labled as `swp`.
one more note: on the e-manual tutorial, it also said there would be `export ROS_MASTER_URI=http://{IP_ADDRESS_OF_REMOTE_PC}:11311` & `export ROS_HOSTNAME={IP_ADDRESS_OF_RASPBERRY_PI_3}` on .bashrc file and we should modify the IP adresses of them but I geuss this as well is for noetic local Image and not on ubuntu 22 raw server. although I add them to bashrc just to be sure, but if I'm not mistaken these are for ROS 1 structure and we could set up the ROS network without these two variables.

that's all. thank you!

# section 2

## SLAM


