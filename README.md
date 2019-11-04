# RogerSkyline-1
Home is where the web server is

This projct is an interesting exercise in commands and dealing with virtual machines to create a web server.

## Setup of Virtual Machine

To start you need to install a debian image to something like VirtualBox, which I used. With the project ther are some very specific aspects. 

- A disk size of 8 GB
- Have at least one 4.2 partition
- Use sudo, with this user, to be able to perform operation requiring special rights.
- We don’t want you to use the DHCP service of your machine. You’ve got to
configure it to have a static IP and a Netmask in \30.

So I set that up, Debian offers that a 8 GB hard disk is a good option so it was not diffcult to create that option. The partition setup was simple enough by creating a new partion, specifing the type, size, and what directory it was tied to.

The first thing after the install wit all of the specfic constraints, I run:
```bash
apt-get install sudo
```
This installs sudo and you can use:
```bash
adduser username sudo
```
But I found that didnt work and I needed to run 'su' and then:
```bash
nano /etc/sudoers
```
which allowed me to manually enter in the username I wanted to become a sudoer.

## Static IP address

Some constraints for the the ports and IP address:

- We don’t want you to use the DHCP service of your machine. You’ve got to
configure it to have a static IP and a Netmask in \30.
-You have to change the default port of the SSH service by the one of your choice.
SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT
be allowed directly, but with a user who can be root.

So to start the project needs for both the host and guest ips to communicate. First we will ant to determine our ip address in the guest. The command:
```bash
ip -br -h -f inet a
```
when ran in the virtual box returns a brief hunman readable family {inet} ip address.

But we also want to check to see if its running and availble in the Host ip:
```bash
arp -a
```

Manually you need to change your IPv4 Network Mask, you do so by going
> File -> Host Network Manager

You need both your guest ip which is your virtual machine, and your host ip to be the same with port on both machines opened. This is done manually in the 
> Settings -> Network -> Adapter 1 -> Advanced -> Portforwarding
of VirtualBox.

<img width="826" alt="Screen Shot 2019-11-04 at 1 38 08 PM" src="https://user-images.githubusercontent.com/22520221/68163644-db478300-ff0f-11e9-8f7f-1c06bf169f49.png">



## Connect via SSH and enter into the virtual machine without a password

For this part, reader I am going to assume you know about SSH and RSA keys. When used with the virtual machine RSA public keys make for a secure and password free way to authenicate. You will need to generate a key and then copy it.
```bash
ssh-copy-id -i id_rsa.pub 'vm-username'@'ip-address' -p '2000'
```
I set my port to 2000, as seen before, but its really what ever you want that works. The ip address wil also vary. Until you copy this, in your sshd_configue VM file PasswordAuthenication should be set to yes. Afterwards it can be changed to no. 


## Keeping up to Date and Services

- It will also have to be up to date as well as the whole packages installed to meet
the demands of this subject.
- Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.
- Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.
- Stop the services you don’t need for this project.
