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
3. Open a terminal in the same directory as the downloaded file (Go to the folder with the downloaded kernel (for istance Download folder), right click on the window and select open in terminal otherwise manually use ```cd``` command in a generic terminal). 
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




Then you can finish your installation with:

sudo update-initramfs -u
sudo update-grub

ERROR NO WIFI MODULE WITH KERNEL 5.15.170:
lanciare il comando: 
sudo dmesg | grep iwl
dovrebbero comparire dei messggi di errore che indicano le versioni firmware che il sistema ha provato a caricare. Verificare che in /lib/firmware effettivamente non ci siano i file indicati. 
Guardare i firmware version che il sistema cerca di caricare (nel mio caso era 7625D-29.ucode) dopo di che andare su https://wireless.docs.kernel.org/en/latest/en/users/drivers/iwlwifi/core_release.html#core-release poi nella sezione "core release" cliccare sulla versione che serve e verrà scaricata. Poi andare in Download, tasto destro, open in terminal:  
sudo cp iwlwifi-7265D-29.ucode /lib/firmware
poi fare il reboot del sistema e ora dovrebbe funzioare

(nel caso fosse utile con questo comando è possibile controllare il network contrller :
lspci -nn | grep Network)

https://wireless.wiki.kernel.org/en/use … rs/iwlwifi 
 
INSTALL ETHERCAT MASTER v1.6
la base è questa guida ma ci sono alcune modifiche
https://icube-robotics.github.io/ethercat_driver_ros2/quickstart/installation.html
Andare creare una cartella in download o da qualche parte (esempio io ho creato una cartella "app" in "home" e metto tutto li quando devo scaricare delle app from source) 
fare tasto destro, open in terminal :

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git autoconf libtool pkg-config make build-essential net-tools
git clone https://gitlab.com/etherlab.org/ethercat.git

cd ethercat

git checkout stable-1.6

sudo rm /usr/bin/ethercat

sudo rm /etc/init.d/ethercat

./bootstrap  # to create the configure script

./configure --disable-8139too --enable-generic --enable-r8169 --disable-eoe --prefix=/opt/etherlab

make all modules

sudo make modules_install install

sudo depmod

sudo ln -s /opt/etherlab/bin/ethercat /usr/bin/

sudo ln -s /opt/etherlab/etc/init.d/ethercat /etc/init.d/ethercat

sudo mkdir -p /etc/sysconfig

sudo cp /opt/etherlab/etc/sysconfig/ethercat /etc/sysconfig/ethercat

sudo gedit /etc/udev/rules.d/99-EtherCAT.rules

sudo gedit /etc/sysconfig/ethercat
In the configuration file specify the mac address of the network card to be used and its driver
(in un altro terminale lanciare lshw -class network )
MASTER0_DEVICE="ff:ff:ff:ff:ff:ff"  # mac address
DEVICE_MODULES="generic"


poi salvare e chiudere e lanciare 
sudo /etc/init.d/ethercat start
dovrebbe visualizzare: 
Starting EtherCAT master 1.6.2  done
Dopo di che se sono collegati degli slave questi possono essere visulaizzati con: 
ethercat slaves
Per interrompere il master 
sudo /etc/init.d/ethercat stop


Install Armadillo (last stable 14.2.1)
http://codingadventures.org/2020/05/24/how-to-install-armadillo-library-in-ubuntu/

sudo apt install libopenblas-dev liblapack-dev
sudo apt install cmake libarpack2-dev libsuperlu-dev


https://arma.sourceforge.net/download.html


open a terminal in /usr/local:
sudo ln -s /opt/etherlab/bin/ethercat bin/
sudo cp -r /opt/etherlab/etc/* etc/
sudo cp -r /opt/etherlab/share/* share/
sudo cp -r /opt/etherlab/sbin/* sbin/
sudo cp -r /opt/etherlab/lib/* lib/
sudo cp -r /opt/etherlab/include/* include/

ERRORE (error while loading shared libraries: libethercat.so.1: cannot open shared object file: No such file or directory) runnando ros2 run cdpr_alars_2_0 main 
risolto con il comando "$ sudo ldconfig"




``` 
sudo apt install
```
