# A dedicated Neutrino guide:
This is a guide to spin up a Neutrinno node with lnd 0.16.0-Beta. I recommended going through all of the security features on the Raspibolt and Digital ocean guide as part of DYOR before you commit more than a few sats on this node. This will give you options to add supposrt for:
+ Watchtower support
- NOSFT locally (prerequisites: Node v18+)
* LNDg (prerequisites: NGNIX, Docker)
- More to come :)


Simply copy/paste the commands below. The $ is used to show comamnds and dont copy anything in (parentheses). My user is joe. Anywhere you see joe, change it to your user. I stopped using $ for individual commands later since i like to copy paste a few lines at a time. 

This guide is based on the Raspibolt guide with some modifications for it to be a Neutrino node. If you need to know everything that is going on you can find the full guide here https://raspibolt.org/guide/lightning/lightning-client.html.

For added security features check out Raspibolt and Digital Ocean Guides.

## Set up Cloud server
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%203.svg)](https://www.digitalocean.com/?refcode=e22779be4678&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

We will spin up the node using Digidal Ocean and create a droplet. 
Use my referral link https://m.do.co/c/e22779be4678. You will get $200 in credits over a 60 day period. We will be using a $7/mos. Droplet. Then enable the Reserved IP for $5/mos.




## Set up user
```
$ adduser joe (Save pw)
$ usermod -aG sudo joe (enter thru, no need to enter info)
$ rsync --archive --chown=joe:joe ~/.ssh /home/joe
```
## Set up basic firewall (UFW) as root user

```
$ ufw app list
$ ufw allow OpenSSH

->Optional: allow for nosft
$ sudo ufw allow 3000/tcp comment ‘nosft’

->Optional: allow for lndg
$ sudo ufw allow 8889/tcp comment 'allow LNDg SSL'

->Required: allow an IP address
$ sudo ufw allow from xxx.xx.xxx.xxx (Replace xxx.xx.xxx.xxx w/your home IP, google what’s my ip)
$ ufw enable
$ ufw status
$ sudo systemctl enable ufw
```

## Verify and Install LND

As user joe we will verify the install. (this is where i stopped using $).

```
sudo su - joe

cd /tmp

sudo apt install ots

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

## Data Directory
Used -p creates the parent directory also `$ sudo chown joe:joe /data` if needed.
```
sudo adduser --disabled-password --gecos "" lnd

sudo adduser joe lnd

sudo mkdir -p /data/lnd  

sudo chown -R lnd:lnd /data/lnd

sudo su - lnd

ln -s /data/lnd /home/lnd/.lnd

ln -s /data/bitcoin /home/lnd/.bitcoin
```

### Create LND wallet password (Nano is text editor, control x to exit nano)
As user “lnd”, create a text file and enter your LND wallet password. Save and exit (ctrl C)

```
nano /data/lnd/password.txt

chmod 600 /data/lnd/password.txt 
```

## Configuration 

Create the LND configuration file and paste the following content below. Make sure to replace XXX.XXX.XXX.XXX with your **_Reserved IP_** to `externalip=` and change `alias=` to your liking. ***Optional*** for WatchTower Support remove # from `Watchtower` and `wtclient.active=true`

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
protocol.wumbo-channels=true
protocol.no-anchors=false
coop-close-target-confs=24

# Watchtower
#wtclient.active=true

# Performance
gc-canceled-invoices-on-startup=true
gc-canceled-invoices-on-the-fly=true
ignore-historical-gossip-filters=1
stagger-initial-reconnect=true
routing.strictgraphpruning=true

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
neutrino.addpeer=mainnet3-btcd.zaphq.io
neutrino.addpeer=mainnet4-btcd.zaphq.io
neutrino.addpeer=lnd.bitrefill.com:18333
neutrino.addpeer=faucet.lightning.community
neutrino.feeurl=https://nodes.lightning.computer/fees/v1/btc-fee-estimates.json
```

## Run lnd

```
lnd
```

Open a second terminal and keep lnd running in the first 
Commands for the second session start with the prompt $2 (which must not be entered).

```
$2 sudo su - lnd
```

```
$2 lncli create (enter password that matches nano /data/lnd/password.txt)  then type “n”, enter, enter)
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

```
$2 exit
```
->Close out of second LND session

Back in your first SSH session with user “lnd”, LND is still running. Stop LND with Ctrl-C.

(check if the wallet is unlocked automatically)
```
lnd
```
You will see something like this:

[INF] LNWL: The wallet has been unlocked without a time limit

[INF] CHRE: LightningWallet opened

## Create LND systemd unit with the following content. Save and exit. Back to Joe user. Enter “exit” if your still in user LND
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

### Install Node (Nosft prerequisite)
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
A couple times a received some errors. Running this ended up helping. Any suggestions are welcomed.
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

## Install Docker (lndg prerequisite)
Install Docker install (for LNDG) Set up and install Docker Engine from Docker’s apt repository. https://docs.docker.com/engine/install/ubuntu/

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


## Install NGINX (lndg prerequisite)

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
https://github.com/cryptosharks131/lndg#docker-installation-requires-docker-and-docker-compose-be-installed

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
LNDg should now be available on port 8889 with your reserved ip at http://ReserveIP:8889

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
# OPTIONAL: remove unused builds and objects
sudo docker system prune -f
```
Reminder to allow ufw firewall to port 8889 and allow traffic for your home IP


