# setup-xahaud-node

One way to set up a xahaud node with working secure websockets.  Assumptions:
- Dedicated VPS or hardware. This approach does not use docker.
- Not sharing with Evernode or other workloads.
- Ubuntu 22.04 (although it should work with 20.04 as well, but I only have experience running xahaud on 22.04).
- You have a domain and have the ability to create Host (A) records and CNAME records in DNS. I am using examples of EXAMPLE.com, don't YOU use that, use your own.
- As long as you have the ability to create DNS records with your domain, you can create any number of host records using subdomains, there is no need to buy multiple domains. For example, domain "EXAMPLE.com" can have records for every evernode you have (evr-node01.EXAMPLE.com, evr-node02.EXAMPLE.com, etc) in addition to anything else (xahl.EXAMPLE.com, some-other-project-on-xdc-network.EXAMPLE.com, etc.).
- You have a regular user with sudo ability. Never recommended to run things as root.
- I've only tested with for wss, although rpc should work with this setup as well.

NOTE! If you don't know what a Host (A) or CNAME record is, or how to create one or both in DNS, see this link: https://gprivate.com/69poh

## Install xahaud
```
wget https://raw.githubusercontent.com/Xahau/mainnet-docker/main/xahaud-install-update.sh
chmod +x xahaud-install-update.sh
sudo ./xahaud-install-update.sh
```

## Install and start nginx
```
sudo apt-get update
sudo apt-get install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx # this is how you verify it's running
```

## Configure firewall (only allow peer-to-peer, never expose rpc or wss direct ports, 80/443 needed for certbot and secure wss)
```
sudo apt-get install ufw -y # May already be installed but not always
sudo ufw allow 21337
sudo ufw allow 'Nginx Full'
sudo ufw enable # Some providers don't enable ufw by default, this will make sure
```

## Get certs to use with the reverse proxy

In DNS, create a host (A) record pointing to your IP address. Create 2 CNAME records pointing to your host record. For example, if your domain is EXAMPLE.com and IP is 111.111.111.111:
- Host record: xahl pointing to 111.111.111.111
- CNAME record: rpc pointing to xahl.EXAMPLE.com
- CNAME record: wss pointing to xahl.EXAMPLE.com

Note: ensure DNS propagation before moving on. Can you ping all three DNS names you just created and get the same IP address back for all?  No?  Don't move on, fix your DNS.

```
sudo apt-get install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

Note: when prompted, provide your real email address to get certificate alerts, and provide all three DNS names above like this:
`xahl.EXAMPLE.com rpc.EXAMPLE.com wss.EXAMPLE.com`

STOP! Some users report that they get prompted to redirect HTTP traffic to HTTPS, removing HTTP access. If so prompted, select 1!
```
1: No redirect - Make no further changes to the webserver configuration.
```

## Configure xahaud specific reverse proxy config
```
cd /etc/nginx/sites-available
sudo wget -O xahau https://raw.githubusercontent.com/go140point6/xahl-info/main/nginx-reverse-proxy
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/xahau /etc/nginx/sites-enabled/
```

STOP!  You must now edit /etc/nginx/sites-available/xahau, remove the dummy values you find there and replace them with your own. You can have as many or as few ALLOW as you want, as long as it's above the DENY ALL.  Look at the whole file, it should be obvious what you need to update.  Find any EXAMPLE.com and update accordingly.

```
sudo nginx -t # ALWAYS use this to test your changes to nginx config.  If it detects syntax errors, go fix before reloading the service
sudo systemctl reload nginx.service
sudo systemctl status nginx.service # again, this is how you verify it up and running
```

## Configure your xahaud config
- edit the file `/opt/xahaud/etc/xahaud.cfg` and change [port_rpc_public] and [port_wss_public] ip=0.0.0.0 to ip=127.0.0.1
```
sudo systemctl restart xahaud.service
```

## Test stuff
```
xahaud server_info
```

Note: look for `"server_state" : "full",` and your xahaud should be working as expected.  If it's `"server_state" : "connected",` STOP and WAIT! Once it's "full", proceed.

```
ping xahl.EXAMPLE.com
ping rpc.EXAMPLE.com
ping wss.EXAMPLE.com
```

Note: all three should resolve to the same IP address.

Install wscat one of two ways:
```
sudo apt-get update
sudo apt-get install node-ws

OR

npm install -g wscat
```

Then test your websocket:
```
wscat -c wss://wss.EXAMPLE.com
```

If you get back `Connected (press CTRL+C to quit)` then you have a working secure websocket. 

Did you get `error: Unexpected server response: 403`? Go back and edit your nginx config file (xahau) and ensure whatever IP address you are testing is include, reload nginx.

Did you get `error: Unexpected server response: 200`? Reboot the system and try again. Maybe server_state is only connected, don't know, but a reboot fixed it for me.

## Update your evernode websocket
- There seems to be two ways of doing this, the official way (starting with 0.8.2 rippled was changed to xahaud):
```
sudo su -
evernode config xahaud wss://wss.EXAMPLE.com
```

Note: May be deprecated information with 0.8.2 --> This will burn and reissue leases and then will hang. You must wait for it to complete, you will see 4x the number of instances you have, then when sure it's done, ctrl-c out of it.  So if you have 9 instances, you will no it's done when you count 36 lines of output from the process.

- the unofficial way:  edit /etc/sashimono/mb-xrpl/mb-xrpl.cfg and replace `"rippledServer": "wss://xahau.network",` with your newly created secure websocket.
- reboot the node (I didn't try this method, I'm only reporting what others have done).

Once either method is complete, if you do `sudo evernode config xahaud` it should output your secure websocket and `sudo evernode status` should come back as active.

## YAY!  You're done... but nope, this isn't set-it-and-forget-it

xahaud will get updated from time to time and if you don't stay up-to-date, you will eventually get amendment blocked and your node will no long submit for you. You must monitor the Xahau project and update when necessary. Find the latest production build and use it with the commands below. As of this writing, the latest build is `2024.3.12-release%2B790`.

- To update, go to your xahaud VPS and:
```
cd /opt/xahaud/downloads/
sudo wget https://build.xahau.tech/2024.3.12-release%2B790
sudo chown xahaud:xahaud 2024.3.12-release+790
sudo chmod 755 2024.3.12-release+790
cd ../bin
sudo ln -vfns /opt/xahaud/downloads/2024.3.12-release+790 xahaud
sudo systemctl restart xahaud.service
xahaud server_info
```

Note: verify xahaud is using the version you expect. In this case, I see `"build_version" : "2024.3.12-release+790",`
