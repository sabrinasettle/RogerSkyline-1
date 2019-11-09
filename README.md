# RogerSkyline-1
## Home is where the web server is

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
- You have to change the default port of the SSH service by the one of your choice.
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

While there disable your DHCP Server option

You need your IPv4 Network Mask set to
```text
255.255.255.252
```
and your IPv4 address should be
```text
192.168.99.1
```
for the local machine thats one but for the machine we are connecting to it means anything above 100 but only to a certain point

You need both your guest ip which is your virtual machine, and your host ip to be the same with port on both machines opened. This is done manually in the 
> Settings -> Network -> Adapter 1 -> Advanced -> Portforwarding
of VirtualBox.

<img width="826" alt="Screen Shot 2019-11-04 at 1 38 08 PM" src="https://user-images.githubusercontent.com/22520221/68163644-db478300-ff0f-11e9-8f7f-1c06bf169f49.png">

> /etc -> network -> interfaces

insert this 
```text
auto enp0s8
iface enp0s8 inet static
	address 192.168.99.101
	netmask 255.255.255.252
```
which is just like what you manually entered before.

### Connect via SSH and enter into the virtual machine without a password

For this part, reader I am going to assume you know about SSH and RSA keys. When used with the virtual machine RSA public keys make for a secure and password free way to authenicate. You will need to generate a key and then copy it.

```bash
ssh-copy-id -i id_rsa.pub 'vm-username'@'ip-address' -p '2000'
```

So that the public and private key have a handshake so that the key will be greeted by a specific lock.

You need to disable root access which is port 22, I changed mine to 420, 42 with a zero just in case you were wondering.
You need to know the parellel port on the local machine, I set my port to 2000, as seen before, but its really what ever you want that works. The ip address wil also vary. Until you copy this, in your sshd_configue VM file PasswordAuthenication should be set to yes. Afterwards it should be changed to no. 

Now I can use this command to access my VM without using Virtualbox:

```text
ssh mrrogerbluesky@192.168.99.1 -p 2000
```



## Building a Firewall

To protect my server I need to set up a firewall, and create defenses against DOS attacks.

-You have to set the rules of your firewall on your server only with the services used
outside the VM.
-You have to set a DOS (Denial Of Service Attack) protection on your open ports
of your VM.

Debian in its undistrub state does not have UFW, so I had to install it:

```bash
sudo apt-get install ufw
```

then update the rules for my port 420

```bash
sudo ufw allow 420/tcp
```
and this command wil be useful for later when I want to connect through to my web server
```bash
sudo ufw allow 80/tcp
```
and
```bash
sudo ufw allow 443/tcp
```
then
```bash 
sudo ufw enable
```
to enable these rules

### Iptables and Protection

I used the built in program *iptables* to access the information I needed. This needs su or sudo access so it would be a bad idea to make this your first step.

```bash
iptables -L
```

which got me this output:

<img width="375" alt="Screen Shot 2019-11-06 at 9 08 54 AM" src="https://user-images.githubusercontent.com/22520221/68320717-54acb600-0075-11ea-81d5-efd5cce86e9b.png">


Pretty cool, now I need to edit the policys to be stricter to protect my baby server.

Simply protecting but allowing for the ssh to still communicate:

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 420 -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
msudo iptables -A OUTPUT -p tcp --sport 420 -m state --state ESTABLISHED -j ACCEPT
```

for the allowing of https and web server communication:
```bash
sudo iptables -A INPUT -i enp0s8 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

sudo iptables -A INPUT -i enp0s8 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
```

for protecting from a DOS attack:
```bash
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 10/minute --limit-burst 50 -j ACCEPT\
```

## Keeping up to Date and Services
From the pdf:

- It will also have to be up to date as well as the whole packages installed to meet
the demands of this subject.
- Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.
- Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.
- Stop the services you don’t need for this project.

Search just services,
```bash
systemctl list-unit-files
```
or search by their status 'enabled', 'masked', 'disabled'

```bash
systemctl list-unit-files --state=enabled
```
```bash
systemctl list-unit-files | grep enabled
```

I also used 
```bash
ps ax
```

I disabled these services:

keyboard-setup.service, rsyslog.service, apt-daily-upgrade.timer, apt-daily.timer, and console-setup.service


## Setting up the Cron job

Cron jobs are used for running tasks at certain times for this project, I need to:

- Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.
- Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.


crontab -e for editing in nano

```text
0 4 * * 1 ./var/update_weekly.sh & 
0 0 * * * bash /var/cron_moniter & 
@reboot bash /var/update_weekly.sh
```
sudo crontab -l checks what you have in the crontab file

thes files referenced in the crontab are in the var folder in my user's workspace.

Also make sure to have the scipts be excuteable, if its not, use chmod +x on the files that aren't. 

### Connection to Web Server and securing with SSL

I can now access my Apache2 server at this address:
http://192.168.99.1:8080/

I edited the orignal Apache welcome 'It works!' site to make a a site that plays a video and has the lyrics to Mr.Blue Sky.

## Getting the SSL working

This was the most frustrating part of this project so far because it seems like most self-signed ssl tutorials were overly complicated and wrong, really.

I found this github page: https://github.com/gde-pass/roger-skyline-1#apache, by another 42 student to have the anwser I was looking for.

To start I need to gerenate a key and a crt:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -subj "/C=FR/ST=IDF/O=42/OU=Project-roger/CN=10.11.200.247" -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

then from here its simpifing and editing already made files

```bash
sudo nano /etc/apache2/conf-available/ssl-params.conf
```
adding 
```text
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder On

Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff

SSLCompression off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"

SSLSessionTickets Off
```
to that file.

then 
```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

to make the file look more like this:
```text
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost
                ServerName 192.168.99.101

                DocumentRoot /var/www/html

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
                SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
                #SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
                #SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key


                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

        </VirtualHost>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

and then 
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

to make it look like: 

```text
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        Redirect "/" "https://192.168.99.101/"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined


</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

then run these guys

```bash
sudo a2enmod ssl
sudo a2enmod headers
sudo a2ensite default-ssl
sudo a2enconf ssl-params
sudo systemctl stop apache2
sudo systemctl start apache2
```
and thats it!

my site has an insecure unverified by a third party self-signed ssl

Note my ports now look like this:

<img width="827" alt="Screen Shot 2019-11-08 at 2 06 08 PM" src="https://user-images.githubusercontent.com/22520221/68513973-4d320c00-0231-11ea-8d94-e1f28744fa4d.png">

and my site can now be reached at:
```text
https://192.168.99.1:8081/
```

##Turning it in

in the local terminal go to wher your VM is stored for me its

```text
VirtaulBox VMs
```
then cd into the projects's VM

and run 

```bash
shasum < 'ProjectName'.vdi > checksum
```

this shasum will change everytime you change something in the enviroment 

## Resources
For this nice readme:
- https://www.makeareadme.com/

Others:
- https://milq.github.io/enable-sudo-user-account-debian/
- https://www.hostinger.com/tutorials/sudo-and-the-sudoers-file/
- https://wiki.debian.org/DebianFirewall
- https://askubuntu.com/questions/795226/how-to-list-all-enabled-services-from-systemctl
- http://crontab.org/
- https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
- https://spyhce.com/blog/using-iptables-instead-ufw-basic-server-setup
- https://www.cyberciti.biz/tips/linux-security.html

My good friend's github:
- https://github.com/SLO42/roger-skyline-1/tree/master/steps
