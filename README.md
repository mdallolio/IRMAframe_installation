# IRMAframe_installation

First, a Debian-based base operating system must be installed on the machine. For this project distribution, Ubuntu 24.04.4 LTS is fully supported and tested. If you are already working on such a platform, you can skip this section, and if you have any hardware limitations. Please note that since this application requires RT features, it is not recommended to run Linux in a virtual machine. READ THIS ENTIRE GUIDE BEFORE STARTING THE INSTALLATION.
## Install Ubuntu 24.04.04 LTS 
1. Download Ubuntu 24.04.4 LTS image from [here](https://ubuntu.com/download/desktop/thank-you?version=24.04.1&architecture=amd64&lts=true).
2. Create a bootable USB stick (for instance using [Rufus](https://rufus.ie/en/) for Windows). It is recommended to use a 12 GB USB flash drive but for Ubuntu 24.04 8 GB are enough. WARNING: If the flash drive is not empty all data stored inside will be lost in the process! So please create a backup of the device before flashing.
3. Install the operating system alongside Microsoft Windows (it is necessary because the majority of drive systems has to be set up in a Windows-based environment). During the installation select the Dual boot option and allocate a certain amount of the computer's memory considering that those xGB allocated will represent the storage capacity for the Liunx system and won't be availabe for Windows (since the allocated storage space defines the maximum storage capacity of the Linux system is recommended to allocate at least 100 GB but 150 GB or more would be better).  
5. Make sure to leave twice the RAM space as swap area and to have only one additional partition as root (/).
6. Download all the necessary updates during installation (better having wired ethernet connection).
There are two guides available here. The first was developed after an update to IgH EtherCAT Master that enabled the use of 6.x Linux kernels. The second was written earlier and uses 5.x Linux kernels. Follow the first guide unless unsolvable problems appear.

## Method 1
Ubuntu 24.04.4 should automatically install with a 6.8 kernel. Unfortunately, another kernel is required to use IgH EtherCAT Master... but which one?
These instructions were written in August 2025. If you are using this guide years later, please check if there are newer links and update this guide as necessary.
Consider the following useful links:
1. IgH EtherCAT Master v1.6.7 supported device drivers [here](https://docs.etherlab.org/ethercat/1.6/doxygen/devicedrivers.html). (Check if newer versions are available, such as 1.7.x.)
2. Long-term supported Linux kernels: [here](https://www.kernel.org/).
3. Expiration dates of long-term support: [here](https://www.kernel.org/category/releases.html).
As of today, 26 August 2025, IgH EtherCAT Master v1.6.7 supports all Ethernet drivers with kernels 6.1, 6.4, and 6.12 (obviously, if you know the EtherCAT driver on your computer, it's enough to choose a kernel in which it is supported). Then check the long-term supported kernels, in this case 6.1 (End of life Dec 2027), 6.6 (EOL Dec 2026) and 6.12 (EOL Dec 2026).
Since the real time patch has been merged in the mainline since kernel 6.3, the only reasonable choice is kernel is 6.12 since it is long term supported, supports all Ethernet drivers and contains the real time patch. The kernel choice shouldn't affect ROS2 Jazzy as it simply requires Ubuntu 24.04.  


## Method 2: Install older the Real-Time Kernel
1. Boot the system to the default Linux kernel.
2. Download linux-5.15.170.tar.xz from [here](https://www.kernel.org/pub/linux/kernel/v5.x/).
3. Download patch-5.15.170-rt81.patch.xz from [here](https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/older/).
4. Open a terminal in the same directory as the downloaded files (go to the folder with the downloaded kernel (e.g. Download folder), right click on the window and select "open in terminal", otherwise go to the same path with the ``cd``` command in a generic terminal). 
5. Execute the following lines of code in this terminal
   ```
   tar xf linux-5.15.170.tar.xz
   cd linux-5.15.170
   xzcat ../patch-5.15.170-rt81.patch.xz | patch -p1
   cp /boot/config-$(uname -r) .config
   sudo apt install flex bison
   sudo apt-get install libncurses-dev libssl-dev libelf-dev
   yes "" | make oldconfig
   ```
6. Set the terminal to full screen and run: 
   ```
   make menuconfig
   ```
   In the DOS-style window you are prompted to, use the keyboard to navigate to General setup -> Preemption Model and select Fully Preemptible Kernel (RT). Then save and exit.
   Next, conclude the installation by compiling the kernel as follows (it will take some minutes):
   ```
   make modules -j$(nproc)
   make -j$(nproc)
   ```
   WARNING: In case of debian-certs error execute the following commands in the same terminal as before:
   ``` 
   scripts/config --disable SYSTEM_REVOCATION_KEYS
   scripts/config --disable SYSTEM_TRUSTED_KEYS
   ```
   Then execute the following command and press Enter if/when the generation of the missing certifications is required:
   ```
   make -j$(nproc)
   ```
7. Complete the installation with the following commands:
   ```
   sudo make modules_install
   sudo make install
   cd /lib/modules/5.15.170-rt81
   sudo find . -name *.ko -exec strip --strip-unneeded {} +
   
   ```
8. The following command will open a documnet (if you don't like gedit, please use any other text editor). You need to find the line COMPRESS=lz4 and modify it with COMPRESS=xz, save and close. This is needed in order to shrink the huge initramfs image of RT kernel 5.xx, and consequently to avoid kernel panic at boot.
   ```
    sudo gedit /etc/initramfs-tools/initramfs.conf
   ```
9. Then you can finish your installation with:
   ```
   sudo update-initramfs -u
   sudo update-grub
   ```
10. You can close the terminal and boot in the newly installed real-time Linux kernel.

### ERROR NO WIFI MODULE AVAILABLE WITH KERNEL 5.15.170:
When installing the 5.15.170 realtime kernel on Ubuntu 24.04 on an old PC, there was a problem that the wifi module was not available, although there were no problems with the wifi connection from other kernels on the same device. If this problem occurs after booting in this newly installed kernel, follow these steps to solve it. 
Open a terminal and execute: 
```
sudo dmesg | grep iwl
```
This should return error messages indicating the firmware versions the system was trying to load. Check the /lib/firmware directory to see if those files are actually missing. Go back to the terminal and note which firmware version the system is trying to load (in my case it was 7625D-29.ucode), then go to [this](https://wireless.docs.kernel.org/en/latest/en/users/drivers/iwlwifi/core_release.html#core-release) site and find the "Core Releases" section, click on the missing firmware to start the download (of course you will need a wired Internet connection, a different PC, or a different kernel to navigate the Internet and download the file). 
Go to the folder with the downloaded file, right click on the window and select "open in terminal": 
```
sudo cp iwlwifi-7265D-29.ucode /lib/firmware
```
Reboot the system and there should be no problem with the wireless module in the 5.15.170 kernel.

(If necessary, this command can be used to check the network controller: ```lspci -nn | grep Network ```)

 
## Install IgH EtherCAT Master v1.6 by Etherlab
The basis of this installation guide can be found [here](https://icube-robotics.github.io/ethercat_driver_ros2/quickstart/installation.html), but some changes have been made. 
Follow these instructions to install the IgH EtherCAT Master: 
1. Boot into the realtime kernel where the EtherCAT master will be used. Note that the EtherCAT master is only available in the kernel where it is installed. (If the EtherCAT Master is installed on "kernel 1" and has to be used on "kernel 2", the installation on "kernel 2" might break the installation on "kernel 1").
2. Open a terminal and follow these instructions. Depending on your target system, you'll need to modify step 6 of this guide. Open a terminal and run:
   ```
   sudo lshw -class network
   ```
   Under the group described as ``Ethernet Interface``, see if your network interface is using the e1000e or r8169 driver, note that, and then note the serial number. You will need the serial number and network driver later. Close the terminal.
3. Create a directory for example in "Download" or in "Home" (I created a directory called "App" inside "Home" where I could collect all the app installations).
4. Open the newly created directory using the file manager, right click on the window and select "open in terminal". Execute:
   ```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get install git autoconf libtool pkg-config make build-essential net-tools
   git clone https://gitlab.com/etherlab.org/ethercat.git
   ```
5. In the same terminal exeute the following commands in order to select the EtherCAT Master version (the default one should already be the 1.6). The ```sudo rm``` commands are introduced to remove potential files from previous installations of the EtherCAT Master. 
   ```
   cd ethercat
   git checkout stable-1.6
   sudo rm /usr/bin/ethercat
   sudo rm /etc/init.d/ethercat
   ```
6. Create a configuration scricpt: 
   ```
   ./bootstrap  # to create the configure script
   ```
   Depending on your network driver (check step 1) execute one of the following two commands.
   For e1000e drivers:
   ```
   ./configure --disable-8139too --enable-generic --enable-e1000e --disable-eoe --prefix=/opt/etherlab
   ```
   while for r8169 drivers:
   ```
   ./configure --disable-8139too --enable-generic --enable-r8169  --disable-eoe --prefix=/opt/etherlab
   ```
7. Continue with the installation:
   ```
   make all modules
   sudo make modules_install install
   sudo depmod
   ```
8. Complete the installation with: 
   ```
   sudo ln -s /opt/etherlab/bin/ethercat /usr/bin/
   sudo ln -s /opt/etherlab/etc/init.d/ethercat /etc/init.d/ethercat
   sudo mkdir -p /etc/sysconfig
   sudo cp /opt/etherlab/etc/sysconfig/ethercat /etc/sysconfig/ethercat
   ```
   Create a new ```udev``` rule with the following command and in case the  ```KERNEL=="EtherCAT[0-9]*", MODE="0666"``` string is missing, paste it in the window. Save and close the file.
   ```
   sudo gedit /etc/udev/rules.d/99-EtherCAT.rules
   ```
   Execute the following command to edit the config file.  
   ```
   sudo gedit /etc/sysconfig/ethercat
   ```
   Edit ```MASTER0_DEVICE="ff:ff:ff:ff:ff:ff"``` with your previously noted serial number and ```DEVICE_MODULES="r8169"``` or ```DEVICE_MODULES="e1000e"```
   according to your network driver. Save and close the file.
10. Close the terminal and open a new terminal to test the Ethercat Master. Run the following command to use the terminal as superuser:
    ```
    sudo -s
    ```
    Insert your password, press Enter and then execute:
    ```
    /etc/init.d/ethercat start
    ```
    The output should be:
    ```
    Starting EtherCAT master 1.6.2  done
    ```
    If you have EtherCAT slaves connected to the PC, you can run the command ``ethercat slaves`` in the same terminal (after ``/etc/init.d/ethercat start``) and a list of slaves should appear.
    To stop the EtehrCAT Master run in the same terminal:
    ```
    /etc/init.d/ethercat stop
    ```
### Error prevention 
To prevent ROS2 from not being able to find the EtherCAT Master Libraries, check if the files in the different directories in ``/opt/etherlab`` have been copied to the corresponding directory in ``/usr/local``. If not, navigate the file system to ``/usr/local`` and right click on the file manager window and select "open in terminal":
```
sudo ln -s /opt/etherlab/bin/ethercat bin/
sudo cp -r /opt/etherlab/etc/* etc/
sudo cp -r /opt/etherlab/share/* share/
sudo cp -r /opt/etherlab/sbin/* sbin/
sudo cp -r /opt/etherlab/lib/* lib/
sudo cp -r /opt/etherlab/include/* include/
sudo ldconfig
```
NOTE: The very last command prevents an error that was encountered when trying to run a correctly built ROS2 node with EtherCAT master library dependencies. The error was "Error while loading shared libraries: libethercat.so.1: Cannot open shared object file: No such file or directory" and was fixed after running ```sudo ldconfig``.


## Install ROS 2 Jazzy Jalisco
The installation of ROS 2 is not strictly related to the realtime kernel. It is possible to install ROS 2 from any kernel, and it will be available for all other kernels as well. It's suggested to navigate to the ``/`` folder (e.g. where the ``home``, ``dev``, ``etc`` etc. folders are located), right click on the window and select "open in terminal". Then follow the instructions [here](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html) to install ROS2 Jazzy Jalisco from deb packages. 
## Use EtherCAT Master Libraries inside a ROS2 node
Create a workspace containing a C++ package following the instructions in [this](https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html) and [this](https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html) tutorials. Then open the CMakeList.txt file inside the C++ package from your editor and add ``find_package(EtherCAT REQUIRED)`` in the ``find_package`` section and then, after the ``add_executable(<target_name> <target_src_path>)`` line (where  ``<target_name>`` and ``<target_src_path>`` are specific for you) add the line ``target_link_libraries(<target_name> EtherLab::ethercat)``; in case this line already exists just add `` EtherLab::ethercat`` before the ``)``. Save and then build the package again from terminal with ``colcon build``. After these operations it is possible to include the ``#include "ecrt.h"`` directive  inside the .cpp or .h files to include the IgH EtherCAT Master functionalities.
If an error occurs because ROS 2 still cannot find the EtherCAT Master libraries, go back to the CMakeList.txt and add the line ``include_directories(/opt/etherlab/include)`` after the ``find_package`` section (this line could be added anyway even if there are no errors, it is just redundant). 



[comment]: <> (Install Armadillo ,last stable 14.2.1)
[comment]: <> (http://codingadventures.org/2020/05/24/how-to-install-armadillo-library-in-ubuntu/)
[comment]: <> (sudo apt install libopenblas-dev liblapack-dev)
[comment]: <> (sudo apt install cmake libarpack2-dev libsuperlu-dev)
[comment]: <> (https://arma.sourceforge.net/download.html)





