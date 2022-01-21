# VCK-190 Board Reminder

This repositories contains some tips and tricks for the new VCK190 board. 



## Index

* Board setup
* software resource
* ubuntu install



## Board  setup 

#### Boot mode jumper

The VCK190 board contains two core and each core can be configure the boot mode independent via dip switch `SW 11` and `SW 1 ` . 

Boot mode switch for Versal core 

| Mode\SW | 1    | 2    | 3    | 4    |
| ------- | ---- | ---- | ---- | ---- |
| SD      | ON   | OFF  | OFF  | OFF  |
| JTEG    |      |      |      |      |



Boot mode switch for the System Control Core

| Mode\SW | 1    | 2    | 3    | 4    |
| ------- | ---- | ---- | ---- | ---- |
| SD      | ON   | OFF  | OFF  | OFF  |
| JTEG    |      |      |      |      |



#### USB Host Jumper

For the USB Type A connector on the board, It can be configure in jumper 1,2 not teh default jumper 2, 3. this can be referenced in the document UG 



## Petalinux Configuration

This part will provide the configuration needed for running the petalinux with custom rootfs (ubuntu 18.04). 

#### Dependency and Source 

Petalinux version: 2021.1

BSP source:   https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2021-1.html (VCK190 BSP)

Kernel: (might be useful if you wanna switch kernel) https://github.com/Xilinx/linux-xlnx (default kernel 5.1.0) 



#### Create the petalinux project

Follow the document: http://10.6.126.120:30000/mlp-sys-sw/petalinux-v2020.2-patch-and-guide to install and configure the project in 2021.1 version, (for reference please take a look at the UG-1144 in this repo). The operation process is similar with minor differenence. 

1. Replace the device tree (optional for d435 camera, config the device as USB host)

    In `{PROJECT_ROOT}/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi` add the following line:

    ```shell
    &usb0{
            status = "okay";
    };
    &dwc3_0{
            status = "okay";
            dr_mode = "host";
    };
    ```

2. clone the `user_custom.cfg` to `{PROJECT_ROOT}/project-spec/meta-user/recipes-kernel/linux/linux-xlnx/` and  modify the `{PROJECT_ROOT}/project-spec/meta-user/recipes-kernel/linux/linux-xlnx_%.bbappend` as 

    ```shell
    SRC_URI += "file://bsp.cfg \
                file://user_custom.cfg \    
                "
    KERNEL_FEATURES_append = " bsp.cfg"
    FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"
    ```
    add the  `file://user_custom.cfg \ ` to the file.

3. go to `petalinux-config`
    
    configure the image follow the VCK190 follow the offical petalinux guide and set the rootfs to /dev/mmclk0p2

4. build the project and package it to the `/image/linux` folder

    ```shell
    petalinux-build
    petalinux-package --boot --plm --psmfw --uboot --dtb --force
    ```

5. Copy the BOOT.BIN, boot.scr and image.ub from `{PETALINUX_PROJECT_ROOT}/image/linux ` to FAT32 partition of the SD card

## Config the ubuntu root

1. Install qemu tool
    ```shell
    sudo apt-get install qemu-user-static
    ```
    Download the ubuntu base from the source

2. Go to the download the Ubuntu rootfs

    ```shell 
    wget http://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/ubuntu-base-18.04.6-base-arm64.tar.gz
    ```

3. Copy the Ubuntu rootfs in to the second partition in the SD card EXT4 partition. 

    ```shell
    sudo tar xfvp  ./ubuntu-base-18.04.3-base-arm64.tar.gz -C /media/<username>/rootfs/
    
    sync
    ```

4. Copy these config files into the sdcard

    ```shell
    sudo cp -av /usr/bin/qemu-aarch64-static /media/<username>/rootfs/usr/bin/
    
    sudo cp -av /run/systemd/resolve/stub-resolv.conf /media/<username>/rootfs/etc/resolv.conf
    ```

5. Create these mount points

    ```shell
    sudo mount --bind /dev/ /media/<username>/rootfs/dev
    sudo mount --bind /proc/ /media/<username>/rootfs/proc
    sudo mount --bind /sys/ /media/<username>/rootfs/sys
    ```

6. Boot the sdcard rootfs using the qemu 

    ```shell
    sudo chroot  /media/<username>/rootfs/
    ```

7. add  a new user and create a password (you can change username and password)

    ```shell
    useradd -G sudo -m -s /bin/bash <username>
    echo <username>:<password> | chpasswd
    ```

8. Install the nessary software on the sdcard rootfs

    ```shell
    apt-get update
    apt-get upgrade
    apt-get update -y
    apt-get -y install systemd
    apt-get -y install dialog perl locales busybox curl yum
    apt-get -y install sudo ifupdown net-tools ethtool udev wireless-tools iputils-ping resolvconf wget apt-utils wpasupplicant nano
    
    apt-get -y install kmod openssh-client openssh-server
    apt install -y build-essential cmake git
    ```

9. Quit the quem by running the “exit” command

10. Perfrom these umount commads

    ```shell
    sudo umount  /media/<username>/rootfs/dev
    sudo umount  /media/<username>/rootfs/proc
    sudo umount  /media/<username>/rootfs/sys
    ```

11. Copy Linux kernel modules generated in the previous part into sdcard. Go to the petalinux project forlder

    ```shell
    #extract the root in petalinux
    cd images/linux
    mkdir tmp
    cp rootfs.tar.gz tmp/
    cd tmp
    tar -zxvf rootfs.tar.gz
    
    # copy to the sb card rootfs
    cd {PETALINUX_PROJECT}images/linux/tmp
    cp -R lib/modules/4.19.0-xilinx-v2021.1/ /media/<user-name>/rootfs/lib/modules/
    ```

12. Copy the device-tree and the dtb to root

    ```shell
    cd {PETALINUX_PROJECT}images/linux/tmp/
    cp -R boot/* /media/<user-name>/rootfs/boot
    ```

#### You are ready to boot up the system.

# Trouble shooting

## sbin/init not found Kernel Panic
This is cause by the kernel cannot found the init file in your root file system, there have two possible way to solve it 

1. reinstall/install systemd
    
    A inproper installation may cause the systemd/systemd.init file not mount to sbin/init cause the systen cannot be boot, you can use chroot in your host to boot the rootfile system (show in the above) and run 

    ```shell 
    apt-get -y install --reinstall systemd
    or 
    apt-get -y install systemd
    ```

2. some file is not mount properly
    
    you may try to mount all root file again by chroot in your host to boot the rootfile system and mount all file again 
    ```shell
    mount -a
    ```

3. install the init dpkg

    This may be useful when using the old ubuntu root releanse (e.g. Ubuntu 14.04 / 16.04) the systemd install may not include the init dpkg and need to install it manually by navigate to the /etc/resolve.conf</br>
    Edit
    ```shell
    namespace server 8.8.8.8
    ```
    and install the dpkg
    ```shell 
    sudo apt-get update
    sudo apt-get install init 
    ``` 

## Network enable
if the network is not conneted in default (ifconfig no ip)
1. config the network config
    ```shell
    sudo nano /etc/network/interfaces
    ```
2. add the following 
    ```shell
    auto eth0
    iface eth0 inet dhcp
    
    auto eth1
    iface eth1 inet dhcp
    
    #auto lo
    #iface lo inet loopback
    ```
3. restart the network 
    ```shell
    sudo  /etc/init.d/networking restart
    ```
    it should work after this command and it no need to reset on each boot
    
4. (optional) reconfig nameserver
    sometime the nameserver may miss(sudo apt-get update will get Err), you may need to add this domain.
    ```shell
    echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null
    ```
    
## sudo not work
root not work:
login as root user

```shell
chmod 0755 /
or
chmod 755 /etc/
```
user not work
```shell
chmod 0755 /
chmod 0755 /usr/

chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo

su user

# try sudo
```

# Remote Desktop Setup 

### Setup the XRDP Server in your linux machine

1. Install mate dasktop environment 
    ```shell 
    sudo apt-get install mate-core mate-desktop-environment mate-notification-daemon
    ```

2. Install xrdp and xorgxrdp 
    ```shell
    sudo apt-get install xrdp xorgxrdp
    ```
3. add the xrdp user to the ssl group 
    ```shell
    sudo adduser xrdp ssl-cert  
    ```
4. Modify config file</br>
    change the config in startwm.sh
    ```shell
    sudo nano /etc/xrdp/startwm.sh
    # add the following phrase at the bottom
    unset D_BUS_SESSION_BUS_ADDRESS
    exec mate-session
    ```
    change the config in sesman.ini
    ```shell 
    sudo nano /etc/xrdp/sesman.ini
    #find the phrase Policy=Default and change it to UBD / UBDI
    Policy=UBDI
    ```
    change the pemission in firewall 
    ```shell
    sudo ufw allow from ${IP_ADDRESS} to any port 3389
    #or
    sudo ufw allow from ${IP_ADDRESS} to any port 3350
    #or
    sudo ufw allow from ${IP_ADDRESS} to any port 5900
    ```
5. restart the xrdp session
    ```shell
    sudo systemctl restart xrdp 
    #or 
    sudo systemctl stop xrdp 
    sudo systemstl start xrdp 
    ```

6. check the status of the xrdp server
    ```shell
    sudo systemctl status xrdp
    ```
### Access the server 
1. add a new user (optional)
    logn as root user 
    ```shell
    sudo su -s
    ```
    add the new user 
    ```shell
    sudo adduser NEW_USERNAME
    ```
    allow user to use sudo cmd (optional)
    ```shell 
    sudo usermod -aG sudo USER_NAME
    ```
2. Steps to access the server:
 
    #### Windows
    1. Open Remote Desktop
    2. type in the server address
    3. click the connect button 
    4. use following config to enter the session
    ```shell
    Session: Xorg
    username: your username
    password: your password 
    ```
    5. you should see the desktop
    
    #### Ubuntu
    1. Open Remmina
    2. Choose RDP connnection type 
    3. Name your profile
    4. Type in the following settings
    ```shell
    Server: 10.6.88.64
    User name: your username
    User password: your password
    Color depth: choose True color (32bpp)
    Enable share folder (optional)
    ```
    * the share folder should be locate at /home/username/thinclient_drives/
    
    5. Save and connect

# Static IP connection 

1. install netplan.io
    ```shell 
    sudo apt-get install netplan.io
    ```
2. generate netplan
    ```shell
    sudo netplan generate 
    ````
3. check the current ip setting 
    ```shell 
    ifconfig 
    #or 
    route 
    ```
4. open the netplan file 
    ```shell
    sudo nano /etc/netplan/01-netcfg.yaml
    ```
    update the file (exmaple) 
    ```shell
    # This file describes the network interfaces available on your system
    # For more information, see netplan(5).
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
         dhcp4: no
         addresses: [10.6.88.7/24]
         gateway4: 0.0.0.0
         nameservers:
           addresses: [8.8.8.8,8.8.4.4]
        eth1:
         dhcp4: no
         addresses: [10.6.88.6/24]
         gateway4: 0.0.0.0    
         nameservers:
           addresses: [8.8.8.8,8.8.4.4]
    ```
5. apply the net plan
    ```shell
    sudo netplan --debug apply
    ```
6. reboot the machine and see the ip is still the same as you set 

# Others
Resource for developing the project:

Vitis AI user guide: https://www.xilinx.com/support/documentation/sw_manuals/vitis_ai/1_3/ug1414-vitis-ai.pdf 

Vitis AI wiki: https://www.xilinx.com/html_docs/xilinx2019_2/vitis_doc/dpu_over.html 

Vitis AI DPU example TRD https://github.com/Xilinx/Vitis-AI/tree/master/dsa/XVDPU-TRD 
