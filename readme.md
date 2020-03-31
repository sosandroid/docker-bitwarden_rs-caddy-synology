# Docker Bitwarden_RS proxified for Synology NAS

A docker-compose ready package to run [Bitwarden_RS](https://github.com/dani-garcia/bitwarden_rs) proxified with [Caddy server](https://github.com/caddyserver/caddy-docker). This setup provides a Bitwarden_RS server with support of websocket notifications.
 
The goal is to keep the Synology NAS system untouched to be upgrade-proof. This is the reason why caddy server is used to enable the websocket notifications. Synology do not allow advanced setting of their Nginx reverse proxy and ports 80/443 are not free. We will use the embedded reverse proxy and forward the request on some other ports. This is the reason why Bitwarden_rs server is not set to use SSL because behind 2 proxies.

Despite this has been made to run on Synology NAS, this should run on other systems with / without minor adaptations.

  * [Documentation](#documentation)
  * [Pre-requisite](#pre-requisite)
    + [Conventions](#conventions)
  * [Installation](#installation)
  * [Setup](#setup)
  * [Startup and Maintenance](#startup-and-maintenance)
    + [Startup](#startup)
    + [Maintenance](#maintenance)
  * [Hardening](#hardening)
  * [To do](#to-do)
  * [Collaboration](#collaboration)

## Documentation

- [Bitwarden_RS wiki](https://github.com/dani-garcia/bitwarden_rs/wiki)
- [Caddy server 2.0 docs](https://caddyserver.com/docs/)

## Pre-requisite

- A Docker compatible Synology NAS
- An up and running Docker package
- A SSH client
- A domain name with Let's Encrypt certs enabled. This part is off-topic here.

### Conventions
As convention, we will use as example the following
- The domain : `bw.yourdomain.com`
- Folder used : `/volumeX/docker/` to be personnalized to your DSM setup

## Installation

1. Download this repo
2. Unzip and review `docker-compose_bitwarden-caddy.yml` settings
3. Copy this repo content to `/volumeX/docker/`

## Setup

You will first need to access the admin page to fine tune the Bitwarden_RS server. Beware, if accessed once, it will be enabled in `/data/config.json` whatever are the Environment variables. You'll need to disable the admin panel from itself.

1. On Synology's DSM GUI
	1. Go to `Settings` > `Application Portal` > `Reverse proxy`
	2. Add a new entry for `HTTPS`
		- Name : Bitwarden entry point
		- Source protocol : `HTTPS`
		- source domain : `bw.yourdomain.com`
		- port : `443`
		- check `HSTS` and `HTTP/2`
		- destination protocol : `HTTP`
		- destination host: `localhost`
		- port : `8080`
		- In _Custom Headers_ tab, click the drop down list next to _add_ button and choose `websockets`
	3. Add a new entry for HTTP - Make sure webstation is running with a dummy page to be served. This is only as fallback.
		- Name : Bitwarden entry point HTTP
		- Source protocol : `HTTP`
		- source domain : `bw.yourdomain.com`
		- port : `80`
		- destination protocol : `HTTP`
		- destination host: `localhost`
		- port : `80`
2. Using a terminal, connect through SSH
	1. Connect your admin account with password
	2. Gain root using `sudo -i` with your admin password
	3. `cd /volumeX/docker/`
	4. `docker-compose -f docker-compose_bitwarden-caddy.yml pull` to load needed images
	5. Ready for a first run : `docker-compose -f docker-compose_bitwarden-caddy.yml up`

If everything goes well, the prompt will let you know the containers are started and wait until a `ctrl + C` is triggered to stop them. Test the accesses and start the Birwarden_RS fine tune at `https://bw.yourdomain.com/admin`. Once finished disable the access to admin panel from itself. 

Do not forget to [install the clients](https://bitwarden.com/#download) for desktops, browers and mobile. Test their connection.

Shutdown the servers issuing a `ctrl + C` in the terminal

## Startup and Maintenance

### Startup
Once setup is finished, you're ready to launche your "_prouction_" server. Review all the settings and  environment varaibles in the `.yml` file. Test it using the same `docker-compose -f docker-compose_bitwarden-caddy.yml up` as previously. If anything goes well, stop them and run as `detached` with the following command.

	`docker-compose -f docker-compose_bitwarden-caddy.yml up -d`

### Maintenance
Upgrade on a regular basis the servers as they continue to evolve on a daily/weekly basis. Run from a terminal the following commands from as `root` from time to time.

````sh
cd /volumeX/docker/
docker-compose -f docker-compose_bitwarden-caddy.yml down
docker-compose -f docker-compose_bitwarden-caddy.yml pull
docker-compose -f docker-compose_bitwarden-caddy.yml up -d
````

In order to keep a clean system, from time to time, use [this tutoriel](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes).

## Hardening

Your Bitwarden_RS instance is now up and running. It is not yet hardened to brute-force attacks. Please, install Fail2ban to avoid loosing your account control. Here a way to run [Fail2Ban](https://github.com/sosandroid/docker-fail2ban-synology) in Docker on Synology NAS

## To do

Modifying `Caddyfile` to filter ip addresses allowed to access `/admin`. Does not work yet on Caddy V2.0 beta20. `ipfilter` directive not supported

## Collaboration
Feel free to propose any optimization through pull requests
