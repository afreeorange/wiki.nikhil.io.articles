`ufw` is a simple wrapper around `iptables` ([which can be rather complicated](https://github.com/afreeorange/iptables)). You can view all the rules the CLI adds in `/etc/ufw/user.rules`

### Adding Rules and Enabling

Some basic stuff for all interfaces.

```bash
# Block everything incoming
ufw default deny incoming

# Allow all outgoing connections
ufw default allow outgoing

# Allow a port
ufw allow 3306

# Show list of apps that have registered themselves with ufw
# These are in /etc/ufw/applications.d
ufw app list

# Enable an app
ufw allow Samba

# Show all added things (don't need a running firewall for this)
ufw show added

# Enable the firewall
ufw enable

# Check (running) firewall's status
ufw status verbose
```

### Named Rules

See `/etc/ufw/applications.d/` for how these work.

You will run `sudo ufw app update` after modifying stuff in that folder.

You can then run `sudo ufw app list` to see the named rules. To enable/disable,

```bash
sudo ufw allow Immich
sudo ufw deny Immich
sudo ufw delete allow Immich
```

### Deleting Rules

```bash
# Get the rule number
ufw status numbered

# Remove the offending rule
ufw delete 3

# Or you can
```

### Denying Things

```bash
# Deny access to a port
ufw deny 22

# Deny a host and subnet
ufw deny from 192.168.1.4
ufw deny from 192.168.1.0/24

# Deny an outgoing connection
ufw deny out 22
```

### Other stuff

```bash
# Allow a port range
ufw allow 8000:8008/tcp
ufw allow 8000:8008/udp

# Allow an IP Address
ufw allow from 192.168.1.19

# Allow access to an interface
ufw allow in on eth1 to any port 80
ufw allow in on eth1 to any port 443

# Allow an IP Address to a specific port
ufw allow from 192.168.1.19 to any port 3306

# Allow an entire subnet
ufw allow from 192.168.1.0/24

# Allow an entire subnet to a specific port
ufw allow from 192.168.1.0/24 to any port 3306
```

### Disabling IPv6

Edit `/etc/default/ufw` and set `IPV6=no`.

### Other Stuff

Note that `ufw` won't block `macvlan` ports for obvious reasons!

