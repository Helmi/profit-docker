## Main Repository for the Profit-Docker project

This project aims to be a template for a Docker Compose Stack to run a full stack Profit-Trailer instance including important Addons.

### What is Profit-Trailer?

Profit Trailer is one of the most famous Crypto Currency Trading Bots. 

### What Addons are supported

Currently these Addons are included and supported:

- PT-Feeder (Market Trend / Config Automation)
- PT-Tracker (simple, local statistics tool)
- PT-Notifcations (Telegram/Discord Notifications for PT)

### What's the Status of the Profit-Docker project?

This project is completely Work in progress and widely untested. Use at your own risk. More information and detailed instructions will follow later. So far only use as long as you know what you're doing.


### Instructions

Install docker on an empty Linux VPS (only tested with Ubuntu 18.04 so far)

```
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
apt update && apt upgrade -y
apt install docker-compose -y
docker network create webgateway
```

Clone this repository into a folder of your convenience

```
cd /opt 
git clone https://github.com/Helmi/profit-docker.git
cd profit-docker
```

Open docker-compose.yml in your favourite editor and look for domain names and email addresses and change them to your liking.

Note that the domain/subdomain you enter needs to point to your server. The email address is used to generate free SSL certificates (Letsencrypt)

**IMPORTANT:** This repo comes with a structure of folders and config files for Profit-Trailer and all addons. Some of them are empty and can stay empty (like the database for PtTracker for example) but they need to be there. It doesn't work if you delete them.

Also of course you need to modify the config files so they carry everything you want them to. 

**Profit-Trailer is available to all addons through `http://profit-trailer:8081`. No other URL will work, especially localhost won't work.**

PT-Feeder also needs to have file-access to the PT config file. This is available to the container through `/profit-trailer``

Remember to fill in all your license keys, api keys and you also need to set an `server.api_token` for Profit-Trailer which you need to put in the addons config files so they are allowed to connect.

As soon as you're finished, startup everything. 

```
docker-compose up -d
```

You can leave out the `-d` option on your first start so it won't detach. You can exit out with ctrl+c but you may have to manually stop some of the containers and remove them afterwards.

Dont' be confused by error messages on startup. Docker doesn't allow to delay the startup of single containers so the addons are complaining about Profit-Trailer being unavailable at start. It takes 20 to 30 seconds for PT to come up fully and the addons will eventually stop complaining then.