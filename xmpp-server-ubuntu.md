# XMPP Server Setup with Ejabberd on Ubuntu

## 1. Installation of Ejabberd

Update the packages and install Ejabberd:

```bash
sudo apt update
sudo apt install ejabberd
```

## 2. Domain Configuration

Edit the Ejabberd configuration file:

```bash
sudo nano /etc/ejabberd/ejabberd.yml
```

Then, add your domain in the hosts section:

```yaml
hosts:
  - "your-domain.com"
```

## 3. Setting up an Admin User

Specify admin users in the `acl` section of the Ejabberd configuration file:

```yaml
acl:
  admin:
     user:
       - "admin@your-domain.com"
```

Register the admin user with ejabberd:

```bash
sudo ejabberdctl register admin your-domain.com password
```

Remember to replace "password" with a strong, secure password for the admin user.

## 4. Certificate Generation with Let's Encrypt

Install Certbot:

```bash
sudo apt-get install certbot
```

Then, run a temporary web server on port 80 to verify your domain:

```bash
sudo certbot certonly --standalone -d your-domain.com
```

Update the configuration file to include the path to the generated certificate:

```yaml
certfiles:
  - "/etc/letsencrypt/live/your-domain.com/fullchain.pem"
  - "/etc/letsencrypt/live/your-domain.com/privkey.pem"
```

## 5. Configure Access to Certificate

Create a new group called certaccess and add the ejabberd user to it:

```bash
sudo groupadd certaccess
sudo usermod -a -G certaccess ejabberd
```

Change the group ownership of the Let's Encrypt directories and set their permissions:

```bash
sudo chgrp -R certaccess /etc/letsencrypt/live/
sudo chgrp -R certaccess /etc/letsencrypt/archive/
sudo chmod -R 750 /etc/letsencrypt/live/
sudo chmod -R 750 /etc/letsencrypt/archive/
```

## 6. Resolve TURN IPv4 Address Warning

Get your public IP address:

```bash
dig +short myip.opendns.com @resolver1.opendns.com
```

Then, add your public IP to the `turn_ipv4_address` option in the `listen:` section of your configuration file:

```yaml
-
  port: 3478
  transport: udp
  module: ejabberd_stun
  certfile: "/etc/letsencrypt/live/your-domain.com/fullchain.pem"
  use_turn: true
  auth_type: user
  auth_realm: "your-domain.com"
  turn_ipv4_address: "YOUR_PUBLIC_IP"
  turn_ipv6_address: "::"
```

## 7. Apparmor Fix (For Ubuntu 22.04)

Place the following into `/lib/systemd/system/ejabberd.service`:

```bash
Description=A distributed, fault-tolerant Jabber/XMPP server
Documentation=https://www.process-one.net/en/ejabberd/docs/
After=epmd.service network.target
Requires=epmd.service

[Service]
Type=forking
User=ejabberd
Group=ejabberd
LimitNOFILE=65536
Restart=on-failure
RestartSec=5
ExecStart=/bin/sh -c '/usr/sbin/ejabberdctl start && /usr/sbin/ejabberdctl started'
ExecStop=/bin/sh -c '/usr/sbin/ejabberdctl stop && /usr/sbin/ejabberdctl stopped'
ExecReload=/bin/sh -c '/usr/sbin/ejabberdctl reload_config'
PrivateTmp=true
ProtectHome=true
ProtectSystem=full
TimeoutSec=300
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

After modifying your `ejabberd.yml` configuration file, restart your ejabberd server:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ejabberd
sudo systemctl status ejabberd
```

## 8. Changing Node Name

First, backup your current database:

```bash
ejabberdctl backup /path/to/your/backup.bck
```

Stop ejabberd:

```bash
sudo ejabberdctl stop
```

Delete the Mnesia database:

```bash
sudo rm -rf /var/lib/ejabberd/*
```

Change the node name in the `ejabberdctl.cfg` configuration file:
The nodename for ejabberd is usually in the format `ejabberd@hostname`. The `hostname` should be the fully qualified domain name (FQDN) of the server machine.

```bash
ERLANG_NODE=ejabberd@hostname
```

Restart ejabberd:

```bash
sudo systemctl restart ejabberd
sudo ejabberdctl status
```

## 9. Add cluster

**Ensure Same Erlang Cookies:** For two nodes to communicate, they need to have the same Erlang cookie. The cookie is found in `/var/lib/ejabberd/.erlang.cookie`. Make sure this file contains the same string on both machine

Restart ejabberd:

```bash
sudo systemctl restart ejabberd
sudo ejabberdctl status
```

Join the new node to the existing cluster:

```bash
ejabberdctl --node ejabberd@Node1 join_cluster ejabberd@Node2
```

Remember to replace Node1 and Node2 with your actual node names. The Erlang node names should correspond to the long hostname
