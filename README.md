# IRMAframe_installation

First of all, a basic Debian based operating system must be installed on the machine. For this project distribution, Ubuntu 24.04.4 LTS is fully supported and tested for. If you are already working on such a platform you can skip this section, and if you have some hardware constraints. Please note that since RT features are required for this application, it is not recommended to run Linux on a virtual machine. READ THE WHOLE GUIDE BEFORE STARTING THE INSTALLATION.
## Install Ubuntu 24.04.04 LTS 
1. Download Ubuntu 16.04.4 LTS image from [here](https://ubuntu.com/download/desktop/thank-you?version=24.04.1&architecture=amd64&lts=true).
2. Create a bootable USB stick (for instance using [Rufus](https://rufus.ie/en/) for Windows). It is recommended to use a 12 GB USB flash drive but for Ubuntu 24.04 8 GB are enough. WARNING: If the flash drive is not empty all data stored inside will be lost in the process! So please create a backup of the device before flashing.
3. Install the operating system alongside Microsoft Windows (it is necessary because the majority of drive systems has to be set up in a Windows-based environment). During the installation select the Dual boot option and allocate a certain amount of the computer's memory considering that those xGB allocated will represent the storage capacity for the Liunx system and won't be availabe for Windows (since the allocated storage space defines the maximum storage capacity of the Linux system is recommended to allocate at least 100 GB but 150 GB or more would be better).  
5. Make sure to leave twice the RAM space as swap area and to have only one additional partition as root (/).
6. Download all the necessary updates during installation (better having wired ethernet connection).
Once Ubuntu is up and running, a real-time kernel must be installed.

## Install the Real-Time Kernel
1. Boot the system in the deafult Linux Kernel.
2. Download patch-5.15.170-rt81.patch.xz  from [here](https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/).
3. Open a terminal in the same directory as the downloaded file (Go to the folder with the downloaded kernel (for istance Download folder), right click on the window and select "open in terminal" otherwise reach the same path with ```cd``` command in a generic terminal). 
4. Execute the following lines of code in this terminal
   ```
   tar xf linux-5.15.170.tar.xz
   cd linux-5.15.170
   xzcat ../patch-5.15.170-rt81.patch.xz | patch -p1
   cp /boot/config-$(uname -r) .config
   sudo apt install flex bison
   sudo apt-get install libncurses-dev libssl-dev libelf-dev
   yes "" | make oldconfig
   ```
5. Put the terminal in full screen and execute: 
   ```
   make menuconfig
   ```
   In the DOS-style window, you are prompted to, use the keyboard to navigate to General setup -> Preemption Model and select Fully Preemptible Kernel (RT). Then save and exit.
   Next, conclude the installation by compiling the kernel as follows (it will take some minutes):
   ```
   make modules -j$(nproc)
   make -j$(nproc)
   ```
   WARNING: In case of debian-certs error execute in the same terminal as before:
   ``` 
   scripts/config --disable SYSTEM_REVOCATION_KEYS
   scripts/config --disable SYSTEM_TRUSTED_KEYS
   ```
   Then execute the following command and press Enter if it is required to generate the missing certifications:
   ```
   make -j$(nproc)
   ```
6. Complete the installation with the following commands:
   ```
   sudo make modules_install
   sudo make install
   cd /lib/modules/5.15.170-rt81
   sudo find . -name *.ko -exec strip --strip-unneeded {} +
   
   ```
7. The following command will open a documnet (if you don't like gedit, please use any other text editor). You need to find the line COMPRESS=lz4 and modify it with COMPRESS=xz, save and close. This is needed in order to shrink the huge initramfs image of RT kernel 5.xx, and consequently to avoid kernel panic at boot.
   ```
    sudo gedit /etc/initramfs-tools/initramfs.conf
   ```
8. Then you can finish your installation with:
   ```
   sudo update-initramfs -u
   sudo update-grub
   ```
9. You can close the terminal and boot in the newly installed real-time Linux kernel.

### ERROR NO WIFI MODULE AVAILABLE WITH KERNEL 5.15.170:
With the installation of the 5.15.170 real-time kernel on UBUntu 24.04 on a old PC a problem emerged as the wifi module was not available despite there were no problem in the wifi connection from other kernels on the same device. In case this problem shows up after booting in this newly installed kernel follow those steps to solve it. 
Open a terminal and execute: 
```
sudo dmesg | grep iwl
```
This sholud provide error messages indicating firmware versions the system tried to load. Verify that in the directory /lib/firmware those files are actually absent. Go back to the terminal and memorize what firmware version the system is trying to load (in my case was the 7625D-29.ucode) then go to [this](https://wireless.docs.kernel.org/en/latest/en/users/drivers/iwlwifi/core_release.html#core-release) site and fin the section "Core Releases", click on the missing firmware to start the download (obviously you will need a cabled internet connection, another PC or a different kernel to navigate the internet and donwload the file). 
Go to the folder with the downloaded file, right click on the window and select "open in terminal": 
```
sudo cp iwlwifi-7265D-29.ucode /lib/firmware
```
Reboot the system and now there shold be no problem with the wifi module in the kernel 5.15.170.

(In case of necessity this command can be used to check the network controller:```lspci -nn | grep Network ```)

 
## Install IgH EtherCAT Master v1.6 by Etherlab
The basis of this installation guide can be found [here](https://icube-robotics.github.io/ethercat_driver_ros2/quickstart/installation.html) but some changes have been made. 
To install IgH EtherCAT Master follow those instructions: 
1. Boot in the real-time kernel where EtherCAT master will be used. Be careful the Ethercat Master will be available only in the kernel in which it is installed. (Moreover if EtherCAT master is installed on kernel 1 and need to be used on kernel 2, the installation on kernel 2 might broke the istallation on kernel 1)
2. Open a terminal and follow the instructions. Depending on your target system, you'll need to change the step 6 of thi guide. As a rule of thumb, if you are using a desktop PC with an Intel processor, you should have an Intel network card available, whereas if you are using a modern laptop, Realtek network cards are more common. Anyway, open a terminal to find out
   ```
   sudo lshw -class network
   ```
   and, under the group described as ```Ethernet interface```, see if your network interface is using e1000e or r8169 driver, note that down and then note down the serial number as well. Serial number and network driver will be required later. Close the terminal.
3. Create a directory (example in "Download" or in "Home" (I created a "App" directory in "Home" in which i could gather all app installations)).
4. Inside this directory right click on the window and select "open in terminal". Here execute:
   ```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get install git autoconf libtool pkg-config make build-essential net-tools
   git clone https://gitlab.com/etherlab.org/ethercat.git
   ```
5. In the same terminal exeute the following commands in order to select the EtherCAT Master verison (the default one should already be the 1.6). The ```sudo rm``` commands are introduced to remove potential file from previous installations of EtherCAT Master. 
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
   Depending on your network driver execute one of the following two commands.
   For e1000e drivers:
   ```
   ./configure --disable-8139too --enable-generic --enable-e1000e --disable-eoe --prefix=/opt/etherlab
   ```
   while for r8169 drivers:
   ```
   ./configure --disable-8139too --enable-generic --enable-r8169  --disable-eoe --prefix=/opt/etherlab
   ```
7. Proceed with the installation:
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
   Create a new ```udev``` rule with the following command and paste ```KERNEL=="EtherCAT[0-9]*", MODE="0666"``` inside teh window. Save and close the file.
   ```
   sudo gedit /etc/udev/rules.d/99-EtherCAT.rules
   ```
   Execute the following command to edit the config file.  
   ```
   sudo gedit /etc/sysconfig/ethercat
   ```
   Edit ```MASTER0_DEVICE="ff:ff:ff:ff:ff:ff"``` with your previuosly noted serial number and ```DEVICE_MODULES="r8169"``` or ```DEVICE_MODULES="e1000e"```
   according to your network driver. Save and close the file.
10. Close the terminal and open a new terminal to test the Ethercat Master. Run the following command to use the terminal as superuser:
    ```
    sudo -s
    ```
    Insert your password, execute:
    ```
    /etc/init.d/ethercat start
    ```
    The output should be:
    ```
    Starting EtherCAT master 1.6.2  done
    ```
    If you have EtherCAT Slaves connected to the PC you can run in teh same termial (after ```/etc/init.d/ethercat start```) the command ```ethercat slaves``` and as result a list of slaves should appear.
    To stop the EtehrCAT Master run in the same terminal:
    ```
    /etc/init.d/ethercat stop
    ```
### Error prevention 
To prevent the impossility for ROS2 to locate EtherCAT Master Libraries check if the files inside the different direcories in /opt/etherlab have been copied to the respective directory in /usr/local. If that is not the case navigate the file system to /usr/local and there right click on window of the file manager and select "open in terminal":
```
sudo ln -s /opt/etherlab/bin/ethercat bin/
sudo cp -r /opt/etherlab/etc/* etc/
sudo cp -r /opt/etherlab/share/* share/
sudo cp -r /opt/etherlab/sbin/* sbin/
sudo cp -r /opt/etherlab/lib/* lib/
sudo cp -r /opt/etherlab/include/* include/
sudo ldconfig
```
NOTE: The last command prevents an error enountered trying to launch a correctly built ROS2 node containing EtherCAT Master libraries dependencies. The error was  "error while loading shared libraries: libethercat.so.1: cannot open shared object file: No such file or directory" and was solved after the running ```sudo ldconfig```.




Install Armadillo (last stable 14.2.1)
http://codingadventures.org/2020/05/24/how-to-install-armadillo-library-in-ubuntu/

sudo apt install libopenblas-dev liblapack-dev
sudo apt install cmake libarpack2-dev libsuperlu-dev


https://arma.sourceforge.net/download.html



