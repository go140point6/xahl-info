# setup-xahaud-node

One way to set up a xahaud node with working secure websockets.  Assumptions:
- Dedicated VPS or hardware. This approach does not use docker.
- Not sharing with Evernode or other workloads.
- Ubuntu 22.04 (although it should work with 20.04 as well, but I only have experience running xahaud on 22.04).
- You have a domain and have the ability to create Host (A) records and CNAME records in DNS. I am using examples of mydomain.com, don't YOU use that, use your own.
- You have a regular user with sudo ability. Never recommended to run things as root.
- I've only tested with for wss, although rpc should work with this setup as well.


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
sudo ufw allow 21337
sudo ufw allow 'Nginx Full'
```

## Get certs to use with the reverse proxy

In DNS, create a host (A) record pointing to your IP address. Create 2 CNAME records pointing to your host record. For example, if you domain is mydomain.com and IP is 111.111.111.111:
- Host record: xah-mainnet01 pointing to 111.111.111.111
- CNAME record: rpc-xah-mainnet pointing to xah-mainnet01.mydomain.com
- CNAME record: wss-xah-mainnet pointing to xah-mainnet01.mydomain.com

Note: ensure DNS propagation before moving on.

```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

Note: when prompted, provide your real email address to get certificate alerts, and provide all three DNS names above like this:
`xah-mainnet01.mydomain.com rpc-xah-mainnet.mydomain.com wss-xah-mainnet.mydomain.com`

## Configure xahaud specific reverse proxy config
```
cd /etc/nginx/sites-available
sudo wget -O xahau https://raw.githubusercontent.com/go140point6/xahl-info/main/nginx-reverse-proxy
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/xahau /etc/nginx/sites-enabled/
```

STOP!  You must now edit /etc/nginx/sites-available/xahau, remove the dummy values you find there and replace them with your own. You can have as many or as few ALLOW as you want, as long as it's above the DENY ALL.  Look at the whole file, it should be obvious what you need to update.  Find any customdomin.com and update accordingly.

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

Note: look for `"server_state" : "full",` and your xahaud should be working as expected.  May be "connected", if just installed. Give it time.

```
ping xah-mainnet01.mydomain.com
ping rpc-xah-mainnet.mydomain.com
ping wss-xah-mainnet.mydomain.com
```

Note: all three should resolve to the same IP address.

```
sudo apt-get update
sudo apt-get install node-ws
wscat -c wss://wss-xah-mainnet.mydomain.com
```

Note: On a clean system this should work, if you have another version of node installed (like if attempting this on an Evernode system) it probably won't install. If you get back `Connected (press CTRL+C to quit)` then you have a working secure websocket. Did you get `error: Unexpected server response: 403`? Go back and edit your nginx config file (xahau) and ensure whatever IP address you are testing is include, reload nginx.

## Update your evernode websocket
- There seems to be two ways of doing this, the official way:
```
sudo su -
evernode config rippled wss://wss-xah-mainnet.mydomain.com
```

Note: This will burn and reissue leases and then will hang. You must wait for it to complete, you will see 4x the number of instances you have, then when sure it's done, ctrl-c out of it.  So if you have 9 instances, you will no it's done when you count 36 lines of output from the process.

- the unofficial way:  edit /etc/sashimono/mb-xrpl/mb-xrpl.cfg and replace `"rippledServer": "wss://xahau.network",` with your newly created secure websocket.
- reboot the node (I didn't try this method, I'm only reporting what others have done).

Once either method is complete, if you do `sudo evernode config rippled` it should output your secure websocket and `sudo evernode status` should come back as active.

## YAY!  You're done... but nope, this isn't set-it-and-forget-it

NOTE: You probably just get the latest script and run it but I was doing something stupid when I tried it and thought it didn't work. I will test it on next update. Here is how you can manually update.

xahaud will get updated from time to time and if you don't stay up-to-date, you will eventually get amendment blocked and your node will no long submit for you. The recommended way is to build your own xahaud but that is beyond the scope of these instructions.
- Goto https://github.com/Xahau/mainnet-docker and star and watch (all activity) the repo. When a new build is available, it will be under "Live build:".  Note the exact build release, you need for the next commands. Below I am using 2024.1.25-release%2B738 which is the latest at the time of this writing, but you must update with whatever is current.
- To update, go to your xahaud VPS and:
```
cd /opt/xahaud/downloads/
sudo wget https://build.xahau.tech/2024.1.25-release%2B738
sudo chown xahaud:xahaud 2024.1.25-release+738
sudo chmod 755 2024.1.25-release+738
cd ../bin
sudo ln -vfns /opt/xahaud/downloads/2024.1.25-release+738 xahaud
sudo systemctl restart xahaud.service
xahaud server_info
```

Note: verify xahaud is using the version you expect. In this case, I see `"build_version" : "2024.1.25-release+738",`