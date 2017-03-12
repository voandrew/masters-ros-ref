# MASTERS PROJECT - ROS - REF

If the Jetson board is already setup please jump to the **Development** section below.

## Jetson TK1/TX1 Setup

### ROS Indigo

```sh
# Add ROS keyserver
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
sudo apt-get update

# Install ROS Indigo
sudo apt-get install ros-indigo-desktop-full

# Setup ROS
rosdep init
rosdep update

echo "source /opt/ros/indigo/setup.bash" >> ~/.bashrc
source ~/.bashrc
sudo apt-get install python-rosinstall

```

### Network Setup

#### Install bridge-utils
```
sudo apt-get update
sudo apt-get install bridge-utils
```

#### Setup the Bridge
> These instructions asumes you have the configuration below

| Interface | Name             | Jetson  IP Address |
| --------- | ---------------- | ------------------ |
| `eth0`    | Pico Station     | 192.168.1.xx       |
| `eth1`    | Lidar Connection | 192.168.1.xx       |
| `br0`     | Bridge           | 192.168.1.2        |


> **NOTE** The IP address of the LiDAR on `eth1` is set statically to `192.168.1.15`. Be sure that the Jetson's IP address is not set to this otherwise LIDAR streaming will fail.


Bridge eth0 and eth1

```sh
sudo brctl addbr br0
sudo brctl addif br0 eth0
sudo brctl addif br0 eth1
```

Bring up the network bridge `br0`
```sh
sudo ifconfig br0 192.168.1.2 netmask 255.255.255.0 up
```

Verify that the bridge is working
```sh
nmap -sP 192.168.1.0/24
```

#### Setup Internet Connectivity

To get Internet connectivity on the Jetson you must set a **default network device** or **default route**. There are two ways to do this: via the Network Manager GUI or via the commandline interface.

##### Setup Default Route via Network Manager GUI

In "Network Connections" dialog box, edit all the interfaces except for the wireless lan interface (`wlan0`).
In the `IPv4` tab of each interface, press on the `Routes...` button. In this window make sure to check the option to **only use the interface for resources on its network**.

##### Setup Default Route via CLI

```sh
ip route list # Use this to see all the possible current routes
ip route del default 
# ip route add default via [gateway] dev [interface]
ip route add default via 10.250.255.254 dev wlan0  # example for SJSU network
```

### Optional Tools

Install `vim`
```
sudo apt-get install vim
```

Install `tmux`. This program is extremely useful in managing multiple shell sessions over SSH or in a single terminal window.
```
sudo apt-get install tmux
```

> TODO: Add sane defaults for `vim` and `tmux`

## Development

### Cloning the repo 

When first cloning this repository you must run the `init.sh` script to properly setup the catkin workspace. This assumes that ROS-Indigo is already installed. 

You will only need to run this step once when first cloning this repository so ROS can set up the proper build folders and scripts for your machine.

```sh
git clone https://github.com/voandrew/masters-ros-ref.git
cd masters-ros-ref
chmod +x init.sh
./init.sh
```

### Notes

In the root directory of the project workspace make sure to source the setup.bash file first.

```sh
cd masters-ros-ref
source devel/setup.bash
```

### Git Repo

For consistency and tidyness, please follow some of the `git` guidelines for this repo:

#### Commit Messages

All commit messages should be in the form of "This commit will ___" where your message is in the blank space. The first letter must be capitalized as well and not end with a period. 

For example: `git commit -m "Add a new feature`, **not** `git commit -m "added a new feature.`

##### Long Messages
Try to keep the commit message short and simple. If you need a longer description, add a **second `-m`** flag to put the longer message there. 

Example:

```
git commit -m "Fix a bug" -m "The bug did so and so and to fix it I did this and that. The bug arose because of a specific reason"
```

### Committing Files
Don't blindly run `git add .` or `git commit --all`. Check `git status` to make sure no garbage files are added to your commit.

The `.gitignore` file will include some common filetypes not to include like compiled python files (`.pyc`), or temporary vim files `.swp`. If the `.gitignore` is missing a file extension, please add to it. 

Also never try to commit generated files/folders. These include the `build` and `devel` directories. Only commit the `src` folder and allow the `catkin*` commands to generate the proper folders/files.

## Troubleshooting Common Issues

### Unable to run `roscore` due to incorrect ROS_IP

Make sure that the environment variable `ROS_IP` is properly set to the Jetson's IP address on the bridge `br0`. View this IP address by running the command `ifconfig` and looking at the IP address for `br0`

Example
> Note: The output below is not real. Your output may be different 

```sh
ifconfig
br0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    inet6 fe80::1097:d471:db27:21d6%en0 prefixlen 64 secured scopeid 0x4
    inet 192.168.1.132 netmask 0xffffff00 broadcast 192.168.1.255
    nd6 options=201<PERFORMNUD,DAD>
    status: active

```

The important section to note is the **`inet 192.168.1.143`**. Use this IP for the ROS_IP like so:

```
export ROS_IP=192.168.1.132
```

You will need to do this for **every terminal** opened, or you can add the line to `.bashrc` 
