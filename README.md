# Traefik + Docker-Compose on your Synology NAS

## Overview

This git repo is meant to be a bit of a sense check / mud map for building out a docker-compose file that works on your local Synology NAS.

This includes Hardware Acceleration support for Plex on the DS218+ models (anything that can handle X265). There are a **LOT** of traps for young players so I've tried to make this as easy as possible if you have the technical skills/knowledge.

I don't provide a warranty for this or that the guide works in any way or isn't just satire about how stupid docker-compose is to configure, cheers

## Why do you want this

- With docker-compose if you back up the relevant config files you can easily stand up a new server in a few minutes
- It makes it super easy to access and manage your services
- This will automatically update docker images so you don't have to manually do it
- It uses ACME to generate SSL certificates
- It hooks into a dynamic DNS service so you don't need to manually update IPs
- It's fun to do this stuff I guess

## Acknowledgements

- Huge props to @thessem helping me debug the plex issues
- Massive thanks to whoever runs kilian.io for this [amazing guide](https://blog.kilian.io/server-setup/)

## Who this guide is for

I'm not trying to be a prick with the below, this is a really complicated thing to set up requiring a lot of different areas of knowledge.

- If you don't know what the word POSIX means this guide probably isn't for you.
- If you're not familiar with Docker-compose this guide probably isn't for you.
- If you can't take a multi gigabyte text file, extract individual words and find the most popular word in the document with the baked in gnutools this guide probably isn't for you.
- If you can't debug a crashing docker process, this guide probably isn't for you

I will not be providing support on this guide, you are on your own, I have already done most of the really frustrating stuff to figure this out.

## Before you start

Before you get started this doc assumes you've done the following:

- Set up a STATIC IP ADDRESS FOR YOUR SYNOLOGY
- Used your router to point port 80 and point 443 to your synology
- Have SSH access to your Synology
- Installed Docker on your Synology
- Install Docker Compose on your Synology (This may or may not come with docker I can't recall)
- Configured the Synology to not run DSM on port 80 - check [Resources](#resources)
- Set up a `rc.d` script to chmod the hardware accelerator on startup so Plex can use it - check [Resources](#resources)
- Have a domain set up with a remote provider that supports [ddclient](https://sourceforge.net/p/ddclient/wiki/Home/) - I personally like Namecheap, this is up to you.

* Set up subdomains for `@`, `traefik`, `sabnzbd`, `plex` and `sonarr` on your root domain

## Getting Started

- You don't need to clone this repo, choose a directory on your NAS in which you're happy to set up everything
- `mkdir sabnzbd`
- `mkdir traefik`
- `touch traefik/acme.json`
- `mkdir sonarr`
- `mkdir ddclient`
- You probably need to make some of the subdirectories or the like in temp and the other folders, make sure every folder referenced in the docker compose file exists on disk
- copy the `SETUP.env` file into `.env` and update as appropriate. For the HTPASSWD bit [generate something here](https://hostingcanada.org/htpasswd-generator/)
- copy the `synology.docker-compose.yml` file to the directory containing the above
- copy the `traefik.toml` file into the traefik directory and modify the ACME record to have your email address etc
- copy the `ddclient.conf` file into the ddclient directory and update as per your needs, please read the comments, this is a really annoying file to configure
- create a new file called `.env` and copy in the contents of `SETUP.env` configuring for your particular situation. Please read the comments
- `docker-compose -f synology.docker-compose.yml up -d`
- See if that works, if it doesn't inspect individual containers and go from there, God Be With You

If everything works you should be able to access sevices on:

- traefik.yourdomain.com
- plex.yourdomain.com (this may or may not work, otherwise just connect to the local IP, I have had this be super flaky but plex will be fine. MAKE SURE YOU TURN ON REMOTE ACCESS AND HARDWARE ACCELERATION)
- sonarr.yourdomain.com
- sabnzbd.yourdomain.com (this won't work, you need to connect to the local port on the synology and add in sabnzbd.yourdomain.com as an allowed hostname. This requirement sucks!)

Also everything should auto-update and just work as expected. For configuring sonarr with sabnzbd, don't do anything weird, just literally point it at the sabnzbd host link (i.e put in the host as sabnzbd) and it should all flow through. If you want to add in other services, make sure to link them and address them by the virtual hostname the link enables.

## Adding other services

- If you want to add other hosts **MAKE SURE** you use the internal port not the exposed external port ie if it's setup as "12345:1234" set your traefik port to '1234' and make sure it's on the same network
- Make sure your labels are the last thing in your YML section for each item

## Resources

### Moving from Port 80

This code is valid as of May 2020, I have no idea if it still works. If you don't understand what it does, don't do it and stop reading this guide.

- `$ sudo sed -i -e 's/80/81/' -e 's/443/444/' /usr/syno/share/nginx/server.mustache /usr/syno/share/nginx/DSM.mustache /usr/syno/share/nginx/WWWService.mustache`
- `$ synoservicecfg --restart nginx`

### Setting up rc.d Script

- `$ sudo echo 'chmod 777 /dev/dri*' > /usr/local/etc/rc.d/plex-hardware-transcode.sh`
- `$ sudo chmod +x /usr/local/etc/rc.d/plex-hardware-transcode.sh`
