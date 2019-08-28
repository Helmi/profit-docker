## Main Repository for the Profit-Docker project

This project aims to be a template for a Docker Compose Stack to run a full stack Profit-Trailer instance including important Addons and including automatic Reverse-Proxy setup including an automatic SSL setup with a free Letsencrypt certificate.

### What is Profit-Trailer?

Profit Trailer is one of the most famous Crypto Currency Trading Bots. 

### What Addons are supported

Currently these Addons are included and supported:

- PT-Feeder (Market Trend / Config Automation)
- PT-Tracker (simple, local statistics tool)
- PT-Notifcations (Telegram/Discord Notifications for PT)
- PT-Defender (Bag-Busting Tool -> [Website](https://www.ptdefender.com/r/helmi)

### Can I only use some of the addons?

Sure, just open the `docker-compose.yml` file in your editor and comment out the "block" of the addon you don't want to use similar to how PT-Defender is commented out by default. 

### What's the state of the Profit-Docker project?

This project is Work in progress and widely untested. Use at your own risk. More information and detailed instructions will follow later. So far only use as long as you know what you're doing.

__Known limitiations/issues:__

- Only one PT-Stack per VPS
- You need to re-login to Profit-Trailer every time you restart the container or server as Sessions will be killed.

Feel free to post issues to this repository and I'll be glad to help fixing them.

## Instructions

### Setup machine

Install docker on an empty Linux VPS (only tested with Ubuntu 18.04 so far)

```
wget -qO- https://get.docker.com/ | sh
apt update && apt upgrade -y
apt autoremove -y
docker network create webgateway
```

Install docker compose manually (do not use the one from the ubuntu repository). Follow the instructions [here](https://docs.docker.com/compose/install/#install-compose)


### Clone repository

Clone this repository into a folder of your convenience

```
cd /opt 
git clone https://github.com/Helmi/profit-docker.git
cd profit-docker
```

### Do the basic setup

Open docker-compose.yml in your favourite editor and look for domain names and email addresses and change them to your liking.

Note that the domain/subdomain you enter needs to point to your server. The email address is used to generate free SSL certificates (Letsencrypt)

**IMPORTANT:** This repo comes with a structure of folders and config files for Profit-Trailer and all addons. Some of them are empty and can stay empty (like the database for PtTracker for example) but they need to be there. It doesn't work if you delete them.

Also of course you need to modify the config files so they carry everything you want them to. 

**Profit-Trailer is available to all addons through `http://profit-trailer:8081`. No other URL will work, especially localhost won't work.**

PT-Feeder also needs to have file-access to the PT config file. This is available to the container through `/profit-trailer``

Remember to fill in all your license keys, api keys and you also need to set an `server.api_token` for Profit-Trailer which you need to put in the addons config files so they are allowed to connect.

As soon as you're finished, startup everything. 

### Run the stack

```
docker-compose up -d
```

You can leave out the `-d` option on your first start so it won't detach. You can exit out with ctrl+c but you may have to manually stop some of the containers and remove them afterwards.

Dont' be confused by error messages on startup. Docker doesn't allow to delay the startup of single containers so the addons are complaining about Profit-Trailer being unavailable at start. It takes 20 to 30 seconds for PT to come up fully and the addons will eventually stop complaining then.


### Special notes on PT-Defender

On the time of writing of this Readme-File, PT-Defender is only available through a deskto GUI. No problem with the Profit-Docker-Stack but you need to be aware of a few things. You need to setup your own subdomain in the docker-compose.yml file as you do for some of the other services. Also the Profit-Trailer Configuration needs a special line in the application.properties. I have already added that line to the template config included in this repo so be aware that you're not supposed to change it, unless you know what you're doing:

```
# For PT-Defender - Profit Trailer doesn't care about that line
server.hostname = profit-trailer
```

it's a line that's exclusively used to instruct PT-Defender, it's not by any means used by Profit-Trailer itself.

As soon as PT-Defender is running you can reach the VNC view of the Linux desktop through your browser using

```
https://your.subdoma.in/?password=yourpassword
```

Your password needs to be set in the docker-compose.yml file where it says

```
   environment:
     - "VNC_PW=ENTERYOURPASSWORDHERE"
```

You can find PT-Defender being installed in `/usr/local/bin` - look it up through the file explorer and doubleclick it. The settings are saved persistantly but you need to manually restart it once the container gets restarted. I'm looking for a better way to handle this on the long run.

### Use git to version your settings

One reason to chose this setup for me was to have an easy way to manage my bots/settings through git. I personally do this through private repositories at Github. As these are only available to paid accounts you might choose another way (Gitlab, your own private git service, local repositories, etc.). Note that you should not push your settings to a public repo as they contain your license and API keys. You don't want the public to see those.

Here are some additional commands that might help in the process:

The RENAME.gitignore file contains a set of files to better ignore if you want to push your settings to a service like Github. It removes all data files, logs etc. so you don't bloat your repository. To use it rename it to .gitignore.

As the files ignore in that file are already added to the repo you need to remove them to have them ignore from now on:

First edit your git information

```
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "MY_NAME@example.com"
```

```
git rm -r --cached .            # remove all files from git repo
git add .                       # all all files back, ignoring the ones in .gitignore
git commit -m ".gitignore fix"  # commit everything back into the repo
```

If you want to push your repo to any service like Github or Gitlab, you might want to remove the existing remote:

```
git remote -v
git remote rm origin
```

Sidenote: If you want your server to remember your github (or any other service's) credentials use this command:

```
git config --global credential.helper cache
```

Now you can push the settings to your private repo

```
git remote add destination https://github.com/your/repo/url.git
git push destination master
```

If you're familiar enough: Of course you can use whatever other name instead of `destination` and of course you also don't need to push to your `master` branch

### Thanks to

- [Elroy](https://github.com/taniman) for Profit-Trailer
- [Mehtadone](https://github.com/mehtadone) for PT-Feeder
- The PTD Team for PT-Defender
- [But4ler](https://github.com/But4ler/docker-ptdefender) for the inspiration and ideas for the PT-Defender Dockerfile 

### Update any of the packages

Currently the packages need to be updated by me first, as the docker images are hosted on docker hub to be readily available at any time and you don't need to build them yourself. I'm working on a way to automatically make them ready as soon as a new build is out.

So if you know the latest version is available you could for example for profit-trailer use the following chain of commands to update:

```
docker pull helmi74/pd-profit-trailer:latest && docker-compose down && docker-compose up -d
```

This pulls the latest profit-trailer image for PD and restarts the stack. All done.

There's also a command that updates all images to their latest versions as long as you use them with the `:latest` tag. Here it is combined with a docker-compose down and up attached:

```
docker images |grep -v REPOSITORY|awk '{print $1}'|xargs -L1 docker pull  && docker-compose down && docker-compose up -d
```

Check the availability of the image versions on the tags page on Docker Hub. Example for Profit-Trailer: https://hub.docker.com/r/helmi74/pd-profit-trailer/tags/
