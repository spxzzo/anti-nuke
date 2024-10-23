# Synchronizing time with Chrony

## :clock3: 5. Synchronizing time with Chrony

{% hint style="info" %}
Because beacon chain relies on accurate times to perform attestations and produce blocks, your computer's time must be accurate to real NTP or NTS time within 0.5 seconds.
{% endhint %}

{% hint style="info" %}
chrony is an implementation of the Network Time Protocol and helps to keep your computer's time synchronized with NTP.
{% endhint %}

## Setting Up chrony

### :hatching\_chick: Installation

To install chrony, type:

```
sudo apt-get install chrony -y
```

To create the `chrony.conf` config file, copy and paste the following:

```bash
cat > $HOME/chrony.conf << EOF
pool time.google.com       iburst minpoll 1 maxpoll 2 maxsources 3
pool ntp.ubuntu.com        iburst minpoll 1 maxpoll 2 maxsources 3
pool us.pool.ntp.org     iburst minpoll 1 maxpoll 2 maxsources 3

# This directive specify the location of the file containing ID/key pairs for
# NTP authentication.
keyfile /etc/chrony/chrony.keys

# This directive specify the file into which chronyd will store the rate
# information.
driftfile /var/lib/chrony/chrony.drift

# Uncomment the following line to turn logging on.
#log tracking measurements statistics

# Log files location.
logdir /var/log/chrony

# Stop bad estimates upsetting machine clock.
maxupdateskew 5.0

# This directive enables kernel synchronisation (every 11 minutes) of the
# real-time clock. Note that it can’t be used along with the 'rtcfile' directive.
rtcsync

# Step the system clock instead of slewing it if the adjustment is larger than
# one second, but only in the first three clock updates.
makestep 0.1 -1
EOF
```

Move the file to `/etc/chrony/chrony.conf`

```bash
sudo mv $HOME/chrony.conf /etc/chrony/chrony.conf
```

Restart chrony in order for config change to take effect.

```
sudo systemctl restart chronyd.service
```

### :robot: Helpful Commands

To see the source of synchronization data.

```
chronyc sources
```

To view the current status of chrony.

```
chronyc tracking
```

### :fire: Bonus Commands

To pick your timezone run the following command:

```
sudo dpkg-reconfigure tzdata
```

Find your region using the simple text-based GUI.

In the event that you are using national system like India's `IST` select:

```
Asia/Kolkata
```

This will be appropriate for all locales in the country (`IST`, `GMT+0530`).
