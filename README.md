# nxt2itsm

Nexthink score to ITSM solutions (reworked with Arie Joose and Alain Bernard)

Displays Nexthink scores per device in HTML.

The purpose of this package is to provide a connector between Nexthink engines and ITSM tooling, like TOPdesk. It is not limited to this ITSM platform, but the initial version is targeted for TOPdesk.

# Requirements

The Appliance on which this tool will be installed has the following hardware requirements:

- 2 CPU Cores
- 2 GB of RAM
- 20 GB of disk space

In term of connectivity, this Appliance needs to have access to:

- API of the Engine Appliance(s) (port 1671 if default was kept)
- Remote Action API of the Portal Appliance

# Usage 

Create a new tab in the ITSM tool and have it display the following link: "https://appliance_fqdn/device/device_name"

Where device_name is a Windows PC name in Nexthink. This must be configured in the ITSM tool.

The pattern in red, yellow and green colors will be derived from a score.xml file that must be available in all Nexthink engines and must be placed in the same folder as the js scripts. There is an example in this package.

The score.xml may contain Nexthink act links.

# Installation

This installation is based upon a clean install of CentOS. CentOS ISO files can be downloaded from here: https://www.centos.org/download/

If you are not familiar with CentOS firewalls, switch it off:

	sudo systemctl disable firewalld
	sudo systemctl stop firewalld

Edit this file as sudo: /etc/selinux/config, and set: selinux=disabled

### For an online installation:

Install the different necessary components:

	sudo yum install epel-release -y
	sudo yum install nodejs -y
	sudo yum install mod_ssl openssl -y
	sudo yum install git -y
	sudo yum install open-vm-tools -y
	sudo yum update -y 
	sudo systemctl reboot

Install pm2 and also download the different files from the git repository:

	sudo npm -g install pm2
	sudo npm -g install express async libxmljs console-stamp request
	sudo npm -g install nexthink-stuff/nxt2itsm

Make sure pm2 will autostart after a reboot:

	sudo pm2 startup systemd

The solution is now installed in /usr/lib/node_modules/nxt2itsm. Go to the configuration section for the next steps.

### For an offline installation:

Copy the offline_yum_packages.tar file in the home directory of your user (i.e. /home/nexthink). Once this is done, run the following commands
	
	mkdir offline_yum_packages
	tar -xvf offline_yum_packages.tar -C offline_yum_packages/
	cd /home/nexthink/offline_yum_packages/
	sudo rpm -Uvh *.rpm

Copy the three following files from the offline_installation folder

	npmbox.npmbox
	node-v48-linux-x64.tar.gz
	pm2.npmbox
	nxt2itsm.npmbox

Create a subdirectory, then copy the npmbox.npmbox file in it

	mkdir /home/nexthink/npmbox
	cd /home/nexthink/npmbox/
	sudo mv /home/nexthink/npmbox.npmbox .
	
Untar the file in the directory

	sudo tar --no-same-owner --no-same-permissions -xvzf npmbox.npmbox

Then install npmbox with the following command

	sudo npm install --global --cache ./.npmbox.cache --optional --cache-min 99999999999 --shrinkwrap false npmbox
	
Once npmbox is install, you can now move to the installation of the other two files

	cd /home/nexthink/
	sudo npmunbox -g pm2.npmbox
	sudo npmunbox -g nxt2itsm.npmbox
	sudo pm2 startup systemd
	
As one of the dependencies need some specific additional files, create a subfolder to extract them, and then copy them at the right location:

	cd /home/nexthink
	mkdir libxmljs
	mv node-v48-linux-x64.tar.gz libxmljs/
	cd libxmljs/
	tar -xvzf node-v48-linux-x64.tar.gz
	sudo mv Release/ /usr/lib/node_modules/nxt2itsm/node_modules/libxmljs

The solution is now installed in /usr/lib/node_modules/nxt2itsm. Go to the configuration section for the next steps.

# Configuration

Most of the configuration will be done in the settings.json file. You can find how the file is structured by accessing the following link: "https://appliance_fqdn/"

Here are some important points:

- The different score files need to be in the same folder as the nxt2itsm.js file (/usr/lib/node_modules/nxt2itsm/) and then listed in the settings.json file.
- The certificates need to be put in place (or created if selfsigned) in the "keys" subfolder. The files in the folder upon installation are simply examples.

You can have 3 potential files in the "keys" folder depending if you are using trusted certificates or self-signed one: 

	cert.pem -> the certificate file
	key.pem -> the associated key
	ca.pem -> the root certificate (in case of trusted certificates)

Note that the cert file should contain the certificate itself as well as the chained intermediate certificates if there are some to allow the application to link the certificate to the root.

Set either "selfsigned" or "trusted" for the "certificates" option in the settings.json depending if you are using self-signed certificates or certificates signed by a trusted Authority.

In case you need to generate self-signed certificate for the Appliance, you can use the following commands:

	cd /usr/lib/node_modules/nxt2itsm/keys/
	sudo rm -f *.pem
	sudo openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem

Make sure to configure the different settings.json parameters. In case of doubt, you can always start the application and check the different settings from the main page https://appliance_fqdn/.

Once the configuration is done, start pm2 and then the scripts from the /usr/lib/node_modules/nxt2itsm folder with the following commands:

	sudo systemctl start pm2-root
	sudo pm2 start refreshClientlist.js
	sudo pm2 start nxt2itsm.js
	sudo pm2 save

The following commands can help for troubleshooting too:

	pm2 list -> show the list of application running under pm2
	pm2 show id -> replace "id" by the id from the list command to see the details of the specific application
	pm2 restart id -> to restart a specific application
