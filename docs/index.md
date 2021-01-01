Server build documentation

This documents a Virtual Private Server holding several test projects and builds. It will consist of a static web site, served via nginx, and hold a number of LXD containers. Each container will be a separate project. If these projects serve html, the VPS static web site will link to directories serving the project html.

The nginx configuration will act as a reverse proxy, forwarding calls to those directories to the appropriate lxd containers.

Therefore creating a new project is simple: create an lxd container and install the project. Alter the VPS host static html to link to the container. Add a proxy location to the nginx configuration.

Removing a project is just reversing the above steps.

The VPS therefore needs to be configured with LXD, nginx, and certificates to serve https.

Further configuration: Add ssh capability to allow myself remote access - via keys rather than passwords.

Install an MQTT server on the VPS, listenning on the briged LXD network, as my projects are likely to communicate via an mqtt broker.

Add git capability, so my projects on github can be easily cloned on the VPS.

The VPS has domains

webparametrics.com
webparametrics.co.uk

VPS from www.ovh.co.uk

Ubuntu 20.04 Server (64-bit version)
38Gbits

IPv4 address is: 51.89.150.251
IPv6 address is: 2001:41d0:0801:2000:0000:0000:0000:212c

Name : vps806299.ovh.net

Manage at:

https://www.ovh.com/manager


step 1 update the VPS

From the ovh kvm console, log in as root

apt-get update
apt-get upgrade

and reboot


step 2 add user bernard, as root on the VPS

adduser bernard


Step 3 From laptop, and any other PC you want, copy SSH keys

ssh-copy-id bernard@51.89.150.251

This will ask for the user bernard password

so now the following works from laptop
ssh bernard@51.89.150.251
or even just
ssh 51.89.150.251

Step 4 Remove password login and root login

On the VPS as root:

/etc/ssh/sshd_config copied to sshd_config.original

in sshd_config ro

Set the line:
PasswordAuthentication yes
to
PasswordAuthentication no

PermitRootLogin yes
to
PermitRootLogin no

Followed by

systemctl restart ssh.service


Step 5, install iptraf-ng

As root on the VPS:

apt-get install iptraf-ng

The command "iptraf-ng" starts a console traffic monitor


Step 6, install nginx

apt-get install nginx

The server has two directories:

/etc/nginx/sites-available

/etc/nginx/sites-enabled

under sites-available you will see 'default', this points to static files
at /var/www/html which contains the file index.nginx-debian.html which is
the nginx welcome screen. Use a browser to check this is served


Step 7, setup lxd on the VPS

usermod --append --groups lxd bernard

bernard will need to log in and out

lxd init

accept defaults

log out completely, log in as bernard, try

lxc list

To show lxc is working. Note, the internal bridge lxdbr0 is:

IPv4 address for lxdbr0: 10.105.192.1

IPv6 address for lxdbr0: fd42:ad1d:ba59:dd49::1


Step 8, set up container acremscope-db

This is the first of my projects, a container serving a database for my Astronomy Centre Remscope. At this moment it is only created to test LXD, the container itself will be populated later.

Note 20.04 is a long tem support distribution, so:

lxc launch ubuntu:20.04 acremscope-db

lxc list

This gives container ip address 10.105.192.252

lxc exec acremscope-db -- /bin/bash

apt-get update

apt-get upgrade

And to setup the container to serve a database, follow repository acremscope-db docs


Step 9, Getting certificate from letsencrypt

Back on the VPS, as root, following instructions at letsencrypt and https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx

snap install core; snap refresh core

apt-get remove certbot

snap install --classic certbot

ln -s /snap/bin/certbot /usr/bin/certbot

certbot --nginx  -d webparametrics.co.uk -d www.webparametrics.co.uk -d webparametrics.com -d www.webparametrics.com

###

Congratulations! You have successfully enabled https://webparametrics.co.uk,
https://www.webparametrics.co.uk, https://webparametrics.com, and
https://www.webparametrics.com


IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/webparametrics.co.uk/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/webparametrics.co.uk/privkey.pem
   Your cert will expire on 2021-03-31. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

###

Try connecting with browser, should get https ok to the domains.

Test automatic renewal

certbot renew --dry-run

Also to see the certbot auto renewal timer, look at

systemctl list-timers


Step 10, Create a mosquitto broker listenning on the bridge port

apt-get install mosquitto

cd /etc/mosquitto/conf.d

create file acremscope.conf, containing the line

listener 1883 10.105.192.1

then

systemctl restart mosquitto



step 11, install git, and clone repositories

Note: git was found to be already installed on the ubuntu server, but local ssh public key needs copying to git

create ssh key

On the VPS, as user bernard

ssh-keygen -t rsa -b 4096 -C "bernie@skipole.co.uk"

copy contents of .ssh/id_rsa.pub to github

clone any required repositories

git clone git@github.com:bernie-skipole/webparametrics.git



step 12, place static files under /var/www/html

On the VPS, copy static html files from the webparametrics repository
and place these under /var/www/html, owned by root

These should now be available by the web.




