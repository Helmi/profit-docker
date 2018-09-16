## Main Repository for the Profit-Docker project

This project aims to be a template for a Docker Compose Stack to run a full stack Profit-Trailer instance including important Addons and including automatic Reverse-Proxy setup including an automatic SSL setup with a free Letsencrypt certificate.

### What is Profit-Trailer?

Profit Trailer is one of the most famous Crypto Currency Trading Bots. 

### What Addons are supported

Currently these Addons are included and supported:

- PT-Feeder (Market Trend / Config Automation)
- PT-Tracker (simple, local statistics tool)
- PT-Notifcations (Telegram/Discord Notifications for PT)

### What's the state of the Profit-Docker project?

This project is completely Work in progress and widely untested. Use at your own risk. More information and detailed instructions will follow later. So far only use as long as you know what you're doing.

__Known limitiations/issues:__

- Only one PT-Stack per VPS
- You need to re-login to Profit-Trailer every time you restart the container or server as Sessions will be killed.

Feel free to post issues to this repository and I'll be glad to help fixing them.

## Instructions

### Setup machine

Install docker on an empty Linux VPS (only tested with Ubuntu 18.04 so far)

```
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
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

Add back your new destination:

```
git remote add destination https://github.com/your/repo/url.git
git push destination master
```

If you're familiar enough: Of course you can use whatever other name instead of `destination` and of course you also don't need to push to your `master` branch