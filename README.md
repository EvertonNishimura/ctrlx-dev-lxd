# ctrlx-dev-lxd
This is an instruction on how to create a development enviroment for ctrlx-sdk using only lxd, instead of a virtual machine or wsl.
This is based on Version: 1.10.6 of ctrlX core, with ubuntu 18.
To reproduce this step you need to create a ctrlx Core virtual instance, with port forwarding and external access. 
To develop on vs code you need the remote extension.

1 - Download the core (core16) snap.
sudo snap download core
(use the instruction on screen to install it)

2 - Download snap lxd ( we use version 3, which is compatible with ubuntu 18)
sudo snap install lxd --channel=3.0/stable
(use the instruction on screen to install it)

3 - Init LXC
sudo lxc init 
(choose all default options on the wizard, but choose to share connection)
Would you like LXD to be available over the network? (yes/no) [default=no]: yes

4 - Create a LXD container with ubuntu 18.
sudo lxc launch ubuntu:18.04 [your container name]
([your container name] is the name of the container, you can change it freely)

5 - check the internal ip of your container
sudo lxc list (you need to know the ip to ssh into this later)

6 - Login into your new container
lxc shell [your container name]

7 - Configure ssh
sudo nano /etc/ssh/sshd_config
Then change the following properties:
	# Authentication:
	LoginGraceTime 120
	PermitRootLogin yes
	StrictModes yes
  PasswordAuthentication no
  
8 - Create a password for the user root
 passwd ( this command will start a prompt to change the password)
 
9 - restart SSH
systemctl restart ssh
systemctl restart sshd

9 - from other terminal you can now ssh into your container (optional but good for testing)
ssh -A -t rexroot@[ip of your virtual core] -p 8022 ssh root@[ip of your container]

10 - open vs code, and create a new remote connection. Edit the config.yaml as following:

Host [ip of your virtual core] 
  HostName [ip of your virtual core] 
  ForwardAgent yes
  User rexroot
  Port 8022

Host [ip of your container]
  Hostname [ip of your container]
  ProxyJump [ip of your virtual core]
  User root
  port 22

11 - open the remote explorer, after prompted for passwords, you should be remotely connected to the container. Open a new terminal on vs code andclone sdk repository with:
git clone https://github.com/boschrexroth/ctrlx-automation-sdk.git
 
12 - install snapcraft
sudo snap install snapcraft --classic

13 - Follow the instruction of ctrlX sdk to create a snap, them
snapcraft --destructive-mode

14 - to send the generated snap to the core, you login into the core (another terminal) and:
sudo lxc file pull [your container name]/root/path/to/your_snap.snap .
This example uses bionic as container name. Alternatively you can download the snap via vscode and send install it with ctrlX core web insterface.

15 - Install and run the snap on your core virtual
sudo snap install your_snap.snap --devmode

Obs.:
After changing anything on any interface on the virtual core, as of 1.10.6, ssh configuration will always default to disabled.
It means the virtual core will refuse to let you ssh into it. To be able to ssh again you need enable Virtual control emulation
on ctrlx works. After a sucessful login on the qemu emulation screen, you can enable temporarily ssh again with:
sudo snap set system service.ssh.disable=false
Unfortunately, i did not find a way to way to make it permanent, meaning you need to do it everytime you need boot your device.
obs.:
You can route to your container with ( don´t forget to add the port to your ctrlx works):
sudo /sbin/iptables -t nat -A PREROUTING -i eth0 -p tcp --dport [host port] -j DNAT --to-destination [container ip]:[port]



