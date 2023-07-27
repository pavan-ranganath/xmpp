# XMPP Server Setup with Ejabberd

This guide explains how to set up an XMPP server using ejabberd on Ubuntu, including domain setup, certificate generation, and resolving common issues.

## Installation of Ejabberd

```bash
sudo apt update
sudo apt install ejabberd
```

## Domain Configuration

Edit the ejabberd configuration file:

```bash
sudo nano /etc/ejabberd/ejabberd.yml
```

Add your domain in the hosts section:

```yaml
hosts:
  - "your-domain.com"
```

## Setting up an Admin User

Ejabberd allows you to specify admin users who have access to administrative functionalities. You can set this up in the `acl` section of your ejabberd configuration file:

```yaml
acl:
  admin:
     user:
       - "admin@your-domain.com"
```

Finally, you will need to register this user with ejabberd. You can do this with the ejabberdctl command-line tool:

```bash
sudo ejabberdctl register admin your-domain.com password
```

Replace "password" with a strong, secure password for the admin user.

## Certificate Generation with Let's Encrypt

First, install Certbot:

```bash
sudo apt-get install certbot
```

This command will run a temporary web server on port 80 to verify your domain, so make sure no other services are running on port 80 at the time.

```bash
sudo certbot certonly --standalone -d your-domain.com
```

Then, update the configuration file to include the path to the generated certificate:

```yaml
certfiles:
  - "/etc/letsencrypt/live/your-domain.com/fullchain.pem"
  - "/etc/letsencrypt/live/your-domain.com/privkey.pem"
```

## Configure Access to Certificate

Create a new group called certaccess:

```bash
sudo groupadd certaccess
sudo usermod -a -G certaccess ejabberd
```

Add the ejabberd user to the certaccess group:

```bash
sudo usermod -a -G certaccess ejabberd
```

Change the group ownership of the Let's Encrypt directories

```bash
sudo chgrp -R certaccess /etc/letsencrypt/live/
sudo chgrp -R certaccess /etc/letsencrypt/archive/
```

Change the permissions of the directories

```bash
sudo chmod -R 750 /etc/letsencrypt/live/
sudo chmod -R 750 /etc/letsencrypt/archive/
```

## Resolve TURN IPv4 Address Warning

You need to get the public IP address of your server. You can do this using the dig command:

```bash
dig +short myip.opendns.com @resolver1.opendns.com
```

This will return your public IP address.

Look for the `listen:` section and add your public IP to the `turn_ipv4_address` option, something like this:

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

Ubuntu 22.04 ejabberd apparmour profile broken (<https://askubuntu.com/questions/1411679/ubuntu-22-04-ejabberd-apparmour-profile-broken>)

Place here: /lib/systemd/system/ejabberd.service

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

After modifying your ejabberd.yml configuration file, don't forget to restart your ejabberd server to apply the changes:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ejabberd
sudo systemctl status ejabberd
```

Here's a list of common ports that might be open in an ejabberd server:

Sure, I can explain the functionality of each port based on the typical usage in an XMPP environment.

1. **Port 5222**: This is the standard port used by XMPP clients to connect to the server. `ejabberd_c2s` stands for client-to-server and handles connections from XMPP clients. `starttls_required` means that it will require clients to upgrade their connection to a secure one using STARTTLS.

2. **Port 5223**: This port is typically used for legacy SSL connections from clients. Modern clients typically use port 5222 with STARTTLS, but some older clients might still use this port.

3. **Port 5269**: This port is used for server-to-server (s2s) connections. The `ejabberd_s2s_in` module handles incoming server-to-server connections. 

4. **Port 5443**: This port is used for various HTTP-based services, including BOSH (Bidirectional-streams Over Synchronous HTTP), which is a transport protocol that emulates a bidirectional stream between two entities (such as a client and a server) by using multiple synchronous HTTP response/request pairs without requiring the use of polling or asynchronous chunking.

5. **Port 5280**: This port is also used for HTTP-based services, but it's usually used for the admin interface (`/admin`) and for ACME protocol challenges for automatic certificate management.

6. **Port 3478**: This port is used for the STUN (Session Traversal Utilities for NAT) and TURN (Traversal Using Relays around NAT) protocols, which are used to help clients behind NAT (Network Address Translation) connect to the server. TURN is specifically used for relaying traffic when direct (peer-to-peer) connection is not possible.

7. **Port 1883**: This port is used by the MQTT (Message Queuing Telemetry Transport) protocol, which is a lightweight publish-subscribe messaging protocol often used for IoT devices. `mod_mqtt` is the ejabberd module that handles MQTT connections.


Please note that these are default ports, and they can be changed in the ejabberd configuration file (usually named `ejabberd.yml`). The actual ports open on your ejabberd server depend on your specific configuration.

OLDNODE=ejabberd@localhost
NEWNODE=xmpp1@chathub-server-001.entradasolutions.com
OLDFILE=/tmp/old.backup
NEWFILE=/tmp/new.backup