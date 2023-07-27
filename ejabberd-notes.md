# Notes

## XMPP Port Overview

1. **Port 5222**: This is the standard port used by XMPP clients to connect to the server. `ejabberd_c2s` stands for client-to-server and handles connections from XMPP clients. `starttls_required` means that it will require clients to upgrade their connection to a secure one using STARTTLS.

2. **Port 5223**: This port is typically used for legacy SSL connections from clients. Modern clients typically use port 5222 with STARTTLS, but some older clients might still use this port.

3. **Port 5269**: This port is used for server-to-server (s2s) connections. The `ejabberd_s2s_in` module handles incoming server-to-server connections.

4. **Port 5443**: This port is used for various HTTP-based services, including BOSH (Bidirectional-streams Over Synchronous HTTP), which is a transport protocol that emulates a bidirectional stream between two entities (such as a client and a server) by using multiple synchronous HTTP response/request pairs without requiring the use of polling or asynchronous chunking.

5. **Port 5280**: This port is also used for HTTP-based services, but it's usually used for the admin interface (`/admin`) and for ACME protocol challenges for automatic certificate management.

6. **Port 3478**: This port is used for the STUN (Session Traversal Utilities for NAT) and TURN (Traversal Using Relays around NAT) protocols, which are used to help clients behind NAT (Network Address Translation) connect to the server. TURN is specifically used for relaying traffic when direct (peer-to-peer) connection is not possible.

7. **Port 1883**: This port is used by the MQTT (Message Queuing Telemetry Transport) protocol, which is a lightweight publish-subscribe messaging protocol often used for IoT devices. `mod_mqtt` is the ejabberd module that handles MQTT connections.

Please note that these are default ports, and they can be changed in the ejabberd configuration file (usually named `ejabberd.yml`). The actual ports open on your ejabberd server depend on your specific configuration.

## ACL

ACLs specify which users or hosts have privileges. In your case:

- `admin`: The user "<admin@xmpp2.egstech.org>" is given administrative privileges.

- `local`: This is a catch-all rule that matches any user registered on the local ejabberd server.

- `loopback`: This matches any connection coming from the loopback network interfaces, i.e., from the same machine where ejabberd is running.

## API permissions

In the `api_permissions` section, permissions for the ejabberd API are defined. The API commands allow for server control and management, so the permissions are an important part of the configuration to control who can execute which commands and from where.

There are three categories defined:

1. **Console commands**:
   - `from`: The source from which these commands are accepted. In this case, it's `ejabberd_ctl`, which is the command-line administrative tool for ejabberd.
   - `who`: Who can execute these commands. `all` means that any user can execute these commands.
   - `what`: Which commands can be executed. `*` means all commands are allowed.

2. **Admin access**:
   - `who`: Who can execute these commands.
     - The `access` rule allows execution from addresses that match the `loopback` or `admin` ACL (Access Control List).
     - The `oauth` rule allows execution from addresses that have an OAuth token with the "ejabberd:admin" scope, and match the `loopback` or `admin` ACL.
   - `what`: Which commands can be executed. The rules allow all commands except `stop` and `start`.

3. **Public commands**:
   - `who`: These commands can be executed from any address in the `127.0.0.1/8` IP range, i.e., the local host.
   - `what`: Only `status` and `connected_users_number` commands can be executed.

This section of the configuration is used to ensure that only authorized users can execute certain commands, and that some commands can only be executed from certain locations. This can help to improve the security of the ejabberd server.

## Shapers

In this section of your ejabberd configuration, you are defining "shapers" and "shaper rules". Shapers are used to limit the amount of network traffic for certain types of data or users.

1. **Shapers**:

   - `normal`: The normal shaper limits the data rate to 3000 bytes per second after an initial burst of 20000 bytes.

   - `fast`: The fast shaper has a rate limit of 200000 bytes per second with no defined burst size, so it will use the default one.

2. **Shaper rules**: Shaper rules apply the defined shapers to certain situations or users.

   - `max_user_sessions`: A single user can have at most 10 concurrent sessions.

   - `max_user_offline_messages`: An admin can have up to 5000 offline messages, while other users can have up to 100.

   - `c2s_shaper`: Client-to-server traffic shaping rules. Admins have no restrictions (none), while all other users have the normal rate.

   - `s2s_shaper`: Server-to-server traffic is shaped using the fast rate.

The shapers and shaper rules allow you to manage your server's network traffic and ensure that it runs smoothly even under heavy use. It also allows you to prioritize certain types of data or users (in this case, the admin has fewer restrictions than regular users).

## Access rules

Access rules define who is allowed to use certain ejabberd services or perform certain actions:

- `local`: Access is allowed to local users.

- `c2s`: Client-to-server connections are denied for users in the "blocked" ACL, but all other users are allowed.

- `announce`: Only admins are allowed to send announcements.

- `configure`: Only admins are allowed to perform configuration changes.

- `muc_create`: Only local users are allowed to create Multi-User Chat (MUC) rooms.

- `pubsub_createnode`: Only local users are allowed to create nodes in the Publish-Subscribe system.

- `trusted_network`: Only connections from the loopback network are allowed.

## Modules

1. `mod_adhoc`: This module enables support for the XMPP Ad-Hoc Commands protocol, which is used for providing a generic way to provide commands that a client can execute.

2. `mod_admin_extra`: Provides additional administrative commands for ejabberd. These are available via the `ejabberdctl` command and XMPP Ad-Hoc Commands.

3. `mod_announce`: Allows admins to send server-wide messages.

4. `mod_avatar`: It handles user avatars (user pictures) within XMPP.

5. `mod_blocking`: Implements the XMPP blocking command protocol, which allows a user to block communication with other users.

6. `mod_bosh`: This module enables BOSH (Bidirectional-streams Over Synchronous HTTP) support, which is a transport that allows XMPP to be used over HTTP.

7. `mod_caps`: Implements entity capabilities, which is used to broadcast the features a client supports in an efficient way.

8. `mod_carboncopy`: This is a module for message carbons, which allows a client to control the forwarding of its messages to its other resources.

9. `mod_client_state`: This module enables stream management, which is used for better handling of mobile connections.

10. `mod_configure`: Allows clients to get and set values in the server configuration.

11. `mod_disco`: Implements service discovery, which allows entities to find out information about other entities.

12. `mod_fail2ban`: This module can be used to automatically ban IP addresses that show malicious signs like failed logins.

13. `mod_http_api`: This module offers a basic HTTP API to control and get information from ejabberd.

14. `mod_last`: Handles the 'last activity' stanza in XMPP, used to find out when a user was last active.

15. `mod_mqtt`: Enables MQTT (Message Queuing Telemetry Transport) support, a lightweight messaging protocol often used in IoT environments.

16. `mod_muc`: This module enables support for Multi-User Chat (MUC), a protocol for multi-user text chat.

17. `mod_muc_admin`: Provides additional administrative commands for Multi-User Chat.

18. `mod_offline`: Stores messages sent to offline users.

19. `mod_ping`: Allows XMPP entities to send small amounts of data to each other to test whether the other entity is still there.

20. `mod_pres_counter`: Limits the rate of presence notifications to prevent resource exhaustion.

21. `mod_privacy` & `mod_private`: Implement privacy lists and private storage respectively.

22. `mod_pubsub`: Implements the publish-subscribe pattern in XMPP.

23. `mod_push`: Handles push notifications for mobile clients.

24. `mod_push_keepalive`: Helps mobile clients keep their push services connected.

25. `mod_roster`: Handles the user's contact list.

26. `mod_s2s_dialback`: Implements server-to-server XMPP dialback for secure communication between servers.

27. `mod_shared_roster`: Manages shared rosters which are server-side contact lists that can be made available to all users or a subset of users.

28. `mod_sic`: Provides Server IP Check using an XMPP account.

29. `mod_stream_mgmt`: Implements XEP-0198: Stream Management, which allows XMPP clients to maintain a stable connection even when network connectivity is unreliable.

30. `mod_stun_disco`: Discovery STUN/TURN servers via Service Discovery.

31. `mod_vcard` & `mod_vcard_xupdate`: Handle vCards which are a standard for representing and exchanging personal information, like name, address, work info, etc.

32. `mod_version`: Handles the 'jabber:iq:version' stanza, which is used to find out which XMPP client a user is using and its version number.
