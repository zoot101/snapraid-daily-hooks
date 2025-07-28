---
title: snapraid-daily-service-hook
section: 1
header: User Manual
footer: snapraid-daily-service-hook
---

# NAME

snapraid-daily-service-hook - Hook script for use with snapraid-daily
that stops and starts a list of services.

# DESCRIPTION

This script stops a list of services using **systemd** when the main script
starts, and restarts them when the main script finishes. It is intended to be used
with the start and end hook feature of **snapraid-daily**.

To use it, specify the following in the main script configuration file (**snapraid-daily.conf**),
if already using a start/end hook change **start_hook1** to **start_hook2** etc.

```bash
start_hook1="/usr/bin/snapraid-daily-service-hook"
end_hook1="/usr/bin/snapraid-daily-service-hook"

export service1=sonarr
export service2=lidarr
export service3=radarr
...
...
export serviceN=smbd
```

(Where N is the number of services one wants to stop and later start)

The start and end hook parameter are set to point to the hook script directly. A list of
services is also required to pass into the script to stop and later start. **Note that**
**the use of "export" is important!**

The services must be specified as service1= , service2= .... serviceN= . That is starting at **1** and
going up to **N**, where **N** is the number of services to stop/start. It would be much
easier to specify an array here, but it isn't possible to export arrays in bash sadly.

Note that numbers should not be skipped. For example if service1, service2 and service3 are
given and subsequently service5, service6 and service7 are specified in the config file,
then the latter 3 are ignored. **Up to 20 Services are supported**.

Finally it is a good idea to test the script out on its own before using it with **snapraid-daily**
directly. Do that like so (as root) by calling the hook script exactly how **snapraid-daily** will
call it, after populating **snapraid-daily.conf** with the above information:

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test stopping the services
snapraid-daily-service-hook start

# To test re-starting the services
snapraid-daily-service-hook end
```

## RUNNING AS A NON-ROOT USER

Typically root is required to use the **systemctl** command to stop/start services.

However, if one does not run the main **snapraid-daily** script as root, the hook script is also
written to handle this case through the use of **sudo**.

To use it in this case, add your user to the sudo group like so. If the script is not ran
as root, a check is done that the user is part of the sudoers group.

```bash
usermod -aG sudo username
```

For the main script to run as a non-root user and call the hook script for full automation,
it is required to call the **systemctl** command without a password.

To allow usage of the **systemctl** for your user without a password create a file
in **/etc/sudoers.d** like so:

```bash
# As root do
visudo /etc/sudoers.d/username

# Paste in the following, save and close
username ALL=(root) NOPASSWD:/usr/bin/systemctl
```

Lastly test out the script like before while logged in as the user that will run the
main **snapraid-daily** script.

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test stopping the services
snapraid-daily-service-hook start

# To test re-starting the services
snapraid-daily-service-hook end
```

# SOME NOTES ON CREATING HOOK SCRIPTS

These hook scripts are extremely simple and can be used as an example to create
ones own if desired.

Note that if you want the output of a Start/End Hook script to be included in the email or
custom notification hook text, the main script makes its main logfile (which is used
for the email body) available to any hooks it calls through the use of **export**.

To use this in your own hook pipe the commands to **tee -a $main_logfile** like so:

```bash
echo "Hook Successful" | tee -a $main_logfile
```

# AUTHOR

Mark Finnan <mfinnan101@gmail.com>

# COPYRIGHT

Copyright (C) 2025 Mark Finnan

# SEE-ALSO

snapraid-daily(1), snapraid-daily.conf(1), apprise(1), curl(1), sudo(1), snapraid(1), systemctl(1), systemd.unit(1)


