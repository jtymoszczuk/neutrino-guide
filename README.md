# A Dedicated Neutrino Guide: ⚡️
This is a dedicated guide to help quickly spin up a Neutrino node with lnd 0.16.0-Beta. I recommended going through all of the security features on the Raspibolt and Digital ocean guide as part of DYOR before you commit more than a few sats on this node. Its a WIP 👷🏻‍♂️. This guide will give you options to add more tools:
+ [Watchtower](https://github.com/jtymoszczuk/neutrino-guide#configuration) support
- [NOSFT locally](https://github.com/jtymoszczuk/neutrino-guide#optional-install-nosft) (prerequisites: Node v18+)
* [LNDg](https://github.com/jtymoszczuk/neutrino-guide#optional-install-lndg) (prerequisites: NGNIX, Docker)
- [Balance of Satoshi (BOS)](https://github.com/jtymoszczuk/neutrino-guide#optional-balance-of-satoshi) with  Telegram bot and static channel backup (SCB) (prerequisites: Node v18+)
- More to come 🚀

### Why neutrino:
Neutrino is a great way to familiarize yourself with bitcoin. Since it is self-custody you will be responsible for your keys (mnemonic seed). It's quick and cheap to get started, making it ideal for testing. It's a great way to run Nosft locally. If you run a full node it is commonly used as a watchtower or used for its static clearnet IP in [hybrid mode](https://github.com/TrezorHannes/Dual-LND-Hybrid-VPS). It has also been a primary driver for mobile applications. Check out [Neutrino](https://river.com/learn/terms/n/neutrino/) for more info.

### The guide:
This guide is based on the Raspibolt guide with some modifications for it to be a Neutrino node. This is a quick guide with little commentary and explaination. If you want to know everything that is going on you can find the full guide here at [Raspibolt](https://raspibolt.org/guide/lightning/lightning-client.html).  My user is joe. Anywhere you see joe, change it to your user. Don't copy anything in (parentheses) or the $ you see in the begining of some commands. I left out $ signs in some parts so i can copy the `blocks` faster ⚡️.

For added security features check out [Raspibolt](https://raspibolt.org/guide/raspberry-pi/security.html) and [Digital Ocean](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands) Guides.

## Set up Cloud server
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%203.svg)](https://www.digitalocean.com/?refcode=e22779be4678&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

We will spin up the node using Digital Ocean and create a droplet. 
Use my [referral link here](https://m.do.co/c/e22779be4678). You will get $200 in credits over a 60 day period. We will be using a $7/mos. Droplet. Then enable the Reserved IP for $5/mos.

1. Create a project
2. Click "Create" under drop down click -> "Droplet"
3. For Region choose one close to you; Choose Ubuntu for Image; Choose Basic; Select the cheapest droplet $6 - $7.
4. Under "HostName" name your Droplet
5. Create password or SSH keys
6. "Create Droplet"
7. Click on your droplet once it loads. Next to Reserved IP click `Enable now` and assign it.
8. Click your droplets name to get back to it. Then click "Access" then click "Launch Droplet console"

## Set up user
```
$ adduser joe (Save pw)
$ usermod -aG sudo joe (enter thru, no need to enter info)
$ rsync --archive --chown=joe:joe ~/.ssh /home/joe
```
## Set up basic firewall (UFW) as root user
```
ufw app list
ufw allow OpenSSH
ufw enable
ufw status
sudo systemctl enable ufw
```
For LNDg, Nosft: allow home IP address (reminder dont copy the $, its there so you can stop to make edit).
```
$ sudo ufw allow from xxx.xx.xxx.xxx (Replace xxx.xx.xxx.xxx w/your home IP, google what’s my ip)
```

->Optional: allow for nosft
```
sudo ufw allow 3000/tcp comment ‘nosft’
```
->Optional: allow for lndg
```
sudo ufw allow 8889/tcp comment 'allow LNDg SSL'
```

## Verify and Install LND

As user joe we will verify the install. (this is where i stopped using $).

```
sudo su - joe

cd /tmp

sudo apt install ots
```
```
wget https://github.com/lightningnetwork/lnd/releases/download/v0.16.0-beta/lnd-linux-amd64-v0.16.0-beta.tar.gz

wget https://github.com/lightningnetwork/lnd/releases/download/v0.16.0-beta/manifest-v0.16.0-beta.txt

wget https://github.com/lightningnetwork/lnd/releases/download/v0.16.0-beta/manifest-roasbeef-v0.16.0-beta.sig

wget https://github.com/lightningnetwork/lnd/releases/download/v0.16.0-beta/manifest-roasbeef-v0.16.0-beta.sig.ots

sha256sum --check manifest-v0.16.0-beta.txt --ignore-missing

curl https://raw.githubusercontent.com/lightningnetwork/lnd/master/scripts/keys/roasbeef.asc | gpg --import

gpg --verify manifest-roasbeef-v0.16.0-beta.sig manifest-v0.16.0-beta.txt

ots verify manifest-roasbeef-v0.16.0-beta.sig.ots -f manifest-roasbeef-v0.16.0-beta.sig
```

## Now install
```
tar -xzf lnd-linux-amd64-v0.16.0-beta.tar.gz

sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-amd64-v0.16.0-beta/*

lnd --version
```
-> lnd version 0.16.0-beta commit=v0.16.0-beta

## Data Directory
Used -p creates the parent directory. Change joe to your user.
```
sudo adduser --disabled-password --gecos "" lnd

sudo adduser joe lnd

sudo mkdir -p /data/lnd  

sudo chown joe:joe /data

sudo chown -R lnd:lnd /data/lnd

sudo su - lnd

ln -s /data/lnd /home/lnd/.lnd

ln -s /data/bitcoin /home/lnd/.bitcoin
```

### Create LND wallet password (Nano is text editor, control x to exit nano)
As user “lnd”, create a text file and enter your LND wallet password. Save and exit (ctrl C)

```
nano /data/lnd/password.txt
```
Tighten access privileges and make the file readable only for user “lnd”
```
chmod 600 /data/lnd/password.txt 
```
## Configuration 

Create the LND configuration file and paste the following content below. Make sure to replace `XXX.XXX.XXX.XXX` with your **_Reserved IP_** to `externalip=` and change `alias=` to your liking. ***Optional*** for watchtower support remove # from `Watchtower` and `wtclient.active=true`. more info at the [Raspibolt WatchTower guide](https://raspibolt.org/guide/lightning/lightning-client.html#adding-watchtowers)

```
nano /data/lnd/lnd.conf
```

```
# RaspiBolt: lnd configuration
# /data/lnd/lnd.conf

[Application Options]
alias=YOUR_FANCY_ALIAS
debuglevel=info
maxpendingchannels=5
listen=localhost
externalip=XXX.XX.XXX.XXX 
maxbackoff=20s

# Password: automatically unlock wallet with the password in this file
# -- comment out to manually unlock wallet, and see RaspiBolt guide for more secure options
wallet-unlock-password-file=/data/lnd/password.txt
wallet-unlock-allow-create=true

# Automatically regenerate certificate when near expiration
tlsautorefresh=true
# Do not include the interface IPs or the system hostname in TLS certificate.
tlsdisableautofill=true
# Explicitly define any additional domain names for the certificate that will be created.
# tlsextradomain=raspibolt.local
# tlsextradomain=raspibolt.public.domainname.com

# Channel settings
bitcoin.basefee=1000
bitcoin.feerate=1
minchansize=100000
accept-keysend=true
accept-amp=true
#protocol.wumbo-channels=true
protocol.no-anchors=false
coop-close-target-confs=240

# Watchtower
#wtclient.active=true

# Performance
gc-canceled-invoices-on-startup=true
gc-canceled-invoices-on-the-fly=true
ignore-historical-gossip-filters=1
stagger-initial-reconnect=true
#routing.strictgraphpruning=true
#marks too many channels as zombie channels

# Database
[bolt]
db.bolt.auto-compact=true
db.bolt.auto-compact-min-age=168h

[Bitcoin]
bitcoin.active=1
bitcoin.mainnet=1
#bitcoin.node=bitcoind
bitcoin.node=neutrino

#[tor]
#tor.active=true
#tor.v3=true
#tor.streamisolation=true
# for hybrid
#tor.skip-proxy-for-clearnet-targets=true

[neutrino]
# Mainnet addpeers
neutrino.addpeer=btcd-mainnet.lightning.computer
neutrino.addpeer=mainnet1-btcd.zaphq.io
neutrino.addpeer=mainnet2-btcd.zaphq.io
neutrino.addpeer=lnd.bitrefill.com:18333
neutrino.addpeer=faucet.lightning.community
neutrino.feeurl=https://nodes.lightning.computer/fees/v1/btc-fee-estimates.json
neutrino.validatechannels=false
```

## Run lnd

```
lnd
```

Open a second terminal and keep lnd running in the first terminal session running with lnd.
Commands for the second session start with the prompt $2 (which must not be entered).

```
$2 sudo su - lnd
```
You will enter the password that matches `nano /data/lnd/password.txt` after `lncli create' then type “n”, enter, enter
```
$2 lncli create 
```
You will be given your seed. save in a safe and secure place.

---------------BEGIN LND CIPHER SEED---------------
 1. These      2. are   3. your      4. seed 
 5. words    6. you    7. should    8. save   
 9. them      10. somewhere   11. safe  12. especially 
13. if     14. your  15. are   16. going    
17. to       18. have  19. funds    20. on 
21. this  22. node    23. be      24. safe

---------------END LND CIPHER SEED-----------------

!!!YOU MUST WRITE DOWN THIS SEED TO BE ABLE TO RESTORE THE WALLET!!!

-> Close out of second LND session
```
$2 exit
```

Back in your first SSH session where LND is still running Stop LND with Ctrl-C. As user “lnd” check if the wallet is unlocked automatically.
```
lnd
```
You will see something like this near the top:

[INF] LNWL: The wallet has been unlocked without a time limit

[INF] CHRE: LightningWallet opened

### Create LND systemd unit with the following content. Save and exit. Back to Joe user. Enter “exit” if your still in user LND
```
sudo nano /etc/systemd/system/lnd.service
```

```
# RaspiBolt: systemd unit for lnd
# /etc/systemd/system/lnd.service

[Unit]
Description=LND Lightning Network Daemon
Wants=bitcoind.service
After=bitcoind.service

[Service]

# Service execution
###################
ExecStart=/usr/local/bin/lnd
ExecStop=/usr/local/bin/lncli stop

# Process management
####################
Type=simple
Restart=always
RestartSec=30
TimeoutSec=240
LimitNOFILE=128000

# Directory creation and permissions
####################################
User=lnd

# /run/lightningd
RuntimeDirectory=lightningd
RuntimeDirectoryMode=0710

# Hardening measures
####################
# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

# Use a new /dev namespace only populated with API pseudo devices
# such as /dev/null, /dev/zero and /dev/random.
PrivateDevices=true

# Deny the creation of writable and executable memory mappings.
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target

```
## Lastly, Enable, start and unlock LND
```
sudo systemctl enable lnd
sudo systemctl start lnd
systemctl status lnd
```
->make sure it shows enabled in green. Then reboot.

```
sudo reboot
```
## Create lncli commands
```
ln -s /data/lnd /home/joe/.lnd
sudo chmod -R g+X /data/lnd/data/
sudo chmod g+r /data/lnd/data/chain/bitcoin/mainnet/admin.macaroon
```
After your up and running for a couple minutes you should see a couple peers connect and and sync to chain with this command:
```
lncli getinfo
```

# Optional: Install NOSFT!

### Install Node (Nosft & BOS prerequisite)
```
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt-get update
node -v
npm -v
```
You should be running node-> v18.15.0 and npm-> 9.6.3.

### Install Nosft repo:
```
git clone https://github.com/dannydeezy/nosft.git 
cd nosft
npm install
npm run dev
```
leave $nmp run dev running.

### Run Nosft

While terminal is open and running, enter in your browser: your "Reserved_IP:3000"

### if errors:
A couple times I received some errors. Running this ended up helping.
```
rm -rf nosft
npm cache clean --force
npm install @netlify/esbuild
git clone https://github.com/dannydeezy/nosft.git 
cd nosft
npm install
npm run dev
```

# Optional: Install LNDg

## Install Docker (LNDg prerequisite)
Install Docker (for LNDG). Set up and install Docker Engine from [Docker’s apt repository](https://docs.docker.com/engine/install/ubuntu/).

### Set up the repository
1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
sudo apt-get update
```
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```

2. Add Docker’s official GPG key:
```
sudo mkdir -m 0755 -p /etc/apt/keyrings
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker Engine
1. Update the apt package index:
```
sudo apt-get update
```
2. Install Docker Engine, containerd, and Docker Compose.
    1. To install the latest version, run:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Verify that the Docker Engine installation is successful by running the hello-world image:

```
sudo docker run hello-world
```


### Install Docker compose

```
sudo apt  install docker-compose
```


## Install NGINX (LNDg prerequisite)
[Install Source](https://raspibolt.org/guide/raspberry-pi/security.html#prepare-nginx-reverse-proxyd)

```
$ sudo apt install nginx
```

Create a self-signed SSL/TLS certificate (valid for 10 years)

```
$ sudo openssl req -x509 -nodes -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost" -days 3650
```

NGINX is also a full webserver. To use it only as a reverse proxy, remove the default configuration and paste the following configuration into the nginx.conf file.

```
$ sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```
```
$ sudo nano /etc/nginx/nginx.conf
```

```
user www-data;
worker_processes 1;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
  worker_connections 768;
}

stream {
  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
  ssl_session_cache shared:SSL:1m;
  ssl_session_timeout 4h;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;

  include /etc/nginx/streams-enabled/*.conf;

}
```
Create a new directory for future configuration files

```
$ sudo mkdir /etc/nginx/streams-enabled
```

Test this barebone NGINX configuration

```
$ sudo nginx -t
```
> nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
> nginx: configuration file /etc/nginx/nginx.conf test is successful



## Install LNDg from repo using docker
[Install Source](https://github.com/cryptosharks131/lndg#docker-installation-requires-docker-and-docker-compose-be-installed)

```
git clone https://github.com/cryptosharks131/lndg.git
cd lndg
```
Copy and replace the contents (adjust custom volume paths and user "joe" to LND and LNDg folders) of the docker-compose.yaml with the below
```
nano docker-compose.yaml
```
```
services:
  lndg:
    build: .
    volumes:
      - /home/joe/.lnd:/root/.lnd:ro
      - /home/joe/lndg/data:/lndg/data:rw
    command:
      - sh
      - -c
      - python initialize.py -net 'mainnet' -server '127.0.0.1:10009' -d && supervisord && python manage.py runserver 0.0.0.0:8889
    network_mode: "host"
```

Deploy your docker image: 
```
sudo docker-compose up -d
```
LNDg should now be available on port 8889 with your reserved ip at `http://ReserveIP:8889`

Open and copy the password from output file: 
'''
nano data/lndg-admin.txt
'''
Use the password from the output file and the username '''lndg-admin''' to login

### Updating
```
sudo docker-compose down
sudo docker-compose build --no-cache
sudo docker-compose up -d
```
```
## OPTIONAL: remove unused builds and objects
sudo docker system prune -f
```
Reminder to allow ufw firewall to port 8889 and allow traffic for your home IP

# Optional: Balance of Satoshi
Prerequisite - Node V18+ [Here](https://github.com/jtymoszczuk/neutrino-guide#install-node-nosft-prerequisite)

[Install Source](https://plebnet.wiki/wiki/Umbrel_-_Installing_BoS)
```
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
```
Update path and add new line to the end
```
nano ~/.profile
```
```
PATH="$HOME/.npm-global/bin:$PATH"
```
Save and exit (ctrl + x) 

Update shell
```
. ~/.profile
```
Install Balance of Satoshi. This command can be used to upgrade it as well.
```
npm i -g balanceofsatoshis
```
# Installing Telegram Bot for BOS

[Install Source](https://github.com/ziggie1984/miscellanous/blob/97c4905747fe23a824b6e53dc674c4a571ac0f5c/automation_telegram_bot.md#adding-bos-telegram-bot-to-your-startup-scripts)

Go to Telegram Start chat with @BotFather press `/start /newbot` Decide A bot name NodeAliasNew Decide a bot user name for telegram BotName_bot

You will get a long alphanumeric API KEY for the bot, Note that. You can always retrieve it using `/mybot` with BotFather
BotFather will give you a link to your new bot, click on it and it will take you to your bot.

Now come back to your node. On your SSH session run: bos telegram at first prompt type API key (alpha numeric) received from BotFather.

In Telegram go to your bot by clicking on the link provided in BotFather. Then press Start. Type /connect. You will get a numeric key, type that numeric key on the second prompt in your ssh session with bos telegram.

You should get a connected message in your Telegram Bot as well as on SSH session.

Verify by typing /version in your telegram bot and you should see the version number of bos. If you have issues or have installed the telegram bot before you might have to remove the old api key in the by `nano /home/joe/.bos/telegram_bot_api_key`

To run Telegram bot in backgound login via ssh as user joe.

Change to the following directory: `cd /etc/systemd/system/`

create a so called unit-file: `sudo touch bos-telegram.service`

open file with: `sudo nano bos-telegram.service`

copy the following content into it, change VERBINDUNGSCODE with your own code:
```
# /etc/systemd/system/bos-telegram.service

[Unit]
Description=bos-telegram
Wants=lnd.service
After=lnd.service


[Service] 
ExecStart=/home/bos/.npm-global/bin/bos telegram --use-small-units --connect VERBINDUNGSCODE
User=bos
Restart=always
TimeoutSec=120
RestartSec=30
StandardOutput=null
StandardError=journal

[Install]
WantedBy=multi-user.target 
```
Save file and then type the following command in the terminal: `sudo systemctl enable bos-telegram.service`

Reboot your node `sudo reboot`

Wait until your telegram bot shows the new connection to check whether the service is running properly you can type: `sudo systemctl status bos-telegram.service`. For more Telegram commands check out [Raspibolt guide for Telegram](https://raspibolt.org/guide/bonus/lightning/balance-of-satoshis.html#optional-connect-your-node-to-a-telegram-bot)
