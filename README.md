# Why this fork exists

I have personally had a difficult time getting a local development for magento working. This repository exists as a kind of "total idiot's guide to dockerizing magento", because 1) I ain't that smart, and 2) I forget stuff all the time. Who are you again? For those of us who have to use windows at work, this guide is made especially for you. For those who get to use Linux or Mac, just skip the windows specific bits and you should be fine :)

## Prerequisites

Linux, or Windows PC with WSL installed, and version 2 enabled.

Docker

Git

Optionally, chocolatey, a command line utility for windows that simplifies third party package installation. You could also just run everything in WSL.

## Useful Commands


Start (initially build) containers - should be run from working directory: 

    docker compose up -d
	
	
Stop containers (do this if you have problems, need to restart services, or before turning off or hibernating/suspending your computer)
	Available globally: 
	
	docker stop $(ps -aq)
	
	
Show running docker containers (id’s and metadata)
	Available globally: 
	
	docker ps
	
	
Sign into docker environment (php container) as root (should only need this once)
	Available globally: 
	
	docker exec -it -u root <container id> bash
	
	
Sign into docker environment (php container) as normal user
	Available globally: 
	
	docker exec -it -u root <container id> bash
	
	
Activate wsl/bash (allows use of linux utilities in windows)
	Available globally: 
	
	bash or wsl
	
	
Destroy all containers, delete all cached docker images (docker-compose and all files mounted in /src will be preserved, but internal configs will be lost - nuclear option)
	Available globally, containers must already be stopped: 
	
	docker system prune -a

# Setup, version 2 (faster)

You can use powershell, but I recommend using windows terminal, from the windows store, mainly because of the built-in tab support. Also grab debian, which I’d personally recommend, or ubuntu while you’re there. It’s **very important** that whichever terminal you use, it is *always* being started/run “as administrator” during all docker-related tasks, or else you may run into unexpected behavior (i.e. your containers will not finish building).

Once installed, run 

    wsl --set-default-version 2 

This will update/upgrade the embedded linux kernel to an actual, full linux kernel (minus an init system, kernel only), and *should* apply necessary compatibility patches for interfacing with docker. 

Now install docker. You will probably need to reboot your computer a few times during this process. It’s also a great idea to run the little demo command that docker suggests after it’s set up - if you see something other than “could not connect” at localhost/, you’ve done the docker part of this right! To confirm that docker and WSL can communicate correctly, type the following into powershell: 
	
	wsl -d docker-desktop 

If you find yourself in a bash shell, things are working as intended. Typing exit will return you to powershell.

Next, refer to these guides to make sure mkcert is installed for windows, and is accessible via wsl

https://jitheshkt.medium.com/enable-ssl-on-wsl2-apache-windows-10-bcdfef71024a
https://github.com/markshust/docker-magento/discussions/372

Refer to this repository > https://github.com/markshust/docker-magento

You will also need to create a developer account at marketplace.magento.com 
Once you’re set up, be sure to confirm your email. Next, go to 

    my profile>marketplace>access keys

and generate a key pair. You will need to provide a string of your choosing to serve as a seed for the key generation. The resulting public key is your composer username, and the private key, your password.

Create a working directory, ex local_magento, and cd into it.
Run wsl in powershell, and then, in the bash shell:
	
	sudo apt install curl wget php composer

	curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/onelinesetup | bash -s -- magento.test 2.4.3-p1

Hopefully, this will (and should) finish with no issues. When it’s done, run:

	bin/magento --version

If this returns a magento version, or if running bin/magento resolves correctly, then we’re in good shape. 
You should also be able to see a blank luma page at https://magento.test
All needed files should be accessible now at <project_root>/src/

Then run:

	bin/magento sampledata:deploy
	bin/magento setup:upgrade
Mark has made some scripts to simplify the migration of existing databases into this environment: https://github.com/markshust/docker-magento#existing-projects

THE *NEXT* TIME YOUR COMPUTER IS RESTARTED, CONTAINERS MAY NOT RELOAD

Update: on subsequent startups, cd into working directory, and run bin/start
This will pretty much take care of the nginx conf stuff in one easy step!

*** Also, super important, be sure you stop all containers and close windows docker (stop the process entirely from the system tray) before shutting down your machine, or you may potentially have serious problems, including loss of work***

---------------------------------------------------------------------------------------------------------------------------
Original instructions:
In order to fix this, you’ll need to (per the v1 instructions) log into the php docker container as root, and do the following: 

-apt update && apt upgrade, then apt install sudo
-change user “app” password
-add user app to sudoers file w/ visudo
-rm nginx.conf in /var/www/html
-cp nginx.example.conf to nginx.conf
-cd to /etc/nginx, mkdir conf.d
-cp /var/www/html/nginx.conf conf.d/
-log out of container, docker stop $(docker ps -aq) docker-compose up -d
When logging into admin for the first time, you’ll need to create a user with 
bin/magento admin:user:create
And you’ll need to disable 2FA with
bin/magento module:disable Magento_TwoFactorAuth
bin/magento cache:flush

Seems like there might be a *slight issue with css not being loaded properly via a %template% configuration in luma, console gives mime-type error

---------------------------------------------------------------------------------------------------------------------------

# Setup, version 1
### (older instructions, could be useful if you have issues)

You can use powershell, but I recommend using windows terminal, from the windows store, mainly because of the built-in tab support. Also grab debian, which I’d personally recommend, or ubuntu while you’re there. 
Once installed, run 

    wsl --set-default-version 2 

This will update/upgrade the embedded linux kernel to an actual, full linux kernel (minus an init system, kernel only), and *should* apply necessary compatibility patches for interfacing with docker. 

Now install docker. You will probably need to reboot your computer a few times during this process. It’s also a great idea to run the little demo command that docker suggests after it’s set up - if you see something other than “could not connect” at localhost/, you’ve done the docker part of this right! To confirm that docker and WSL can communicate correctly, type the following into powershell: 
	
	wsl -d docker-desktop 

If you find yourself in a bash shell, things are working as intended. Typing exit will return you to powershell.
Refer to this repository > https://github.com/markshust/docker-magento

You will also need to create a developer account at marketplace.magento.com 
Once you’re set up, be sure to confirm your email. Next, go to 

    my profile>marketplace>access keys

and generate a key pair. You will need to provide a string of your choosing to serve as a seed for the key generation. The resulting public key is your composer username, and the private key, your password.

Create a working directory, ex local_magento, and cd into it.
Run wsl in powershell, and then, in the bash shell:

	curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/onelinesetup | bash -s -- magento.test 2.4.3-p1

Hopefully, this will (and should) finish with no issues. When it’s done, run:

	bin/magento --version

If this returns a magento version, or if running bin/magento resolves correctly, then we’re in good shape. Then run:

	bin/magento sampledata:deploy
	bin/magento setup:upgrade

Next, we’ll be using these guides as reference as we add a project hostname to windows, take care of SSL (which we may not actually need), and finally, configure nginx so that we can actually use our new dev environment:

https://github.com/markshust/docker-magento/discussions/372

https://jitheshkt.medium.com/enable-ssl-on-wsl2-apache-windows-10-bcdfef71024a

Now, as an administrator, open C:\Windows\System32\drivers\etc\hosts, and if there is a line beginning with 127.0.0.1 that is not prepended by an octothorpe “#”, add one to comment it out.
Next, type 

	127.0.0.1		magento.test

On a new line. To clarify, there should be a tab character between .1 and magento.test

We are nearly done. Now list your docker containers with docker ps, and note the container id of the php container. Just like a git commit, you won’t have to type the whole thing out, just the first few unique identifiers. We will now, as root, log into the docker container, with 

	docker exec -it -u root <unique ids> bash

We’re only doing a couple of important things here, installing sudo, adding default user “app” to sudoers file, and setting a password for that user.

	apt install sudo
	passwd app

Set the password for the default user now, it doesn’t have to be secure, since this is just for local dev, you can do “password”, whatever you want and can remember

	visudo

This will open the sudoers file.
In the file, copy/paste the line which says:

	root ALL=(ALL:ALL) ALL

and replace the second “root” with app

	app ALL=(ALL:ALL) ALL

then close and save with “:x”
 
Now you can exit and log back in without root.
  

Log into dev container as default user (app)

	sudo apt update && sudo apt upgrade

	sudo apt install wget libnss3-tools
go to https://github.com/FiloSottile/mkcert/releases, and copy the url for the latest linux amd64 release.
in container terminal, 
	
	wget <pasted url>
	mv mkcert<tab complete> mkcert
	chmod a+x mkcert
	sudo mv mkcert /usr/local/bin/
 
	mkcert magento.test 

replace magento.test with whatever alias you have created in the hostfile for 127.0.0.1 and run 

	mkcert -install

to start using the certificate. 

Now, for the final step. Log into the container if you aren’t already. 
If you are not now in /var/www/html, cd into it. 

	rm nginx.conf
	cp nginx.sample.conf nginx.conf

	sudo su
	cd /etc/nginx
	mkdir conf.d
	cp /var/www/html/nginx.conf conf.d/

Nginx will now need to be restarted, so you can stop the containers with docker stop $(docker ps -aq), and bring them back up with a docker-compose up -d.

Now the environment should be working with a stock magento install! Project files are accessible via the src/ subdirectory of the project working directory!
