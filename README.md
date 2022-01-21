# Robocon Junior Training 2021/2022

## Trouble shooting for ubuntu linux environment note

## sbin/init not found Kernel hang on boot 
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

