---
title: snapraid-daily-hooks
section: 1
header: User Manual
footer: snapraid-daily-hooks
---

# NAME

**snapraid-daily-hooks**

# SYNOPSIS

Collection of hook scripts for use with **snapraid-daily**. These extend the
functionality of **snapraid-daily** to add additional notification methods
or to stop a list of services while the main script is running and restart
them afterwards.

# DESCRIPTION

The following hook scripts are included (more will be added in the future)

These hook scripts are intended to be used with the start and end hook function
of **snapraid-daily**.

* **snapraid-daily-service-hook**

These hook scripts then are intended to add additional notification functionality
to **snapraid-daily**.

* **snapraid-daily-ntfy-hook**    

## snapraid-daily-service-hook

This script stops a list of services using **systemd** when the main script
starts, and restarts them when the main script finishes. It is intended to be used
with the start and end hook feature of **snapraid-daily**.

To use it, specify the following in the main script configuration file (**snapraid-daily.conf**)

```bash
start_hook="/usr/bin/snapraid-daily-service-hook"
end_hook="/usr/bin/snapraid-daily-service-hook"

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

The services must be specified as service1= , service2= .... serviceN= . Starting at 1 and
going up to N, where N is the number of services to stop/start. It would be much
easier to specify an array here, but it isn't possible to export arrays in bash sadly.

Note that numbers should not be skipped. For example if service1, service2 and service3 are
given and subsequently service5, service6 and service7 are specified in the config file,
then the later 3 are ignored. **Up to 20 Services are supported**.

Finally it is a good idea to test the script out on its own before using it with **snapraid-daily**
directly. Do that like so (as root) by calling the hook script exactly how **snapraid-daily** will
call it:

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test stopping the services
snapraid-daily-service-hook start

# To test re-starting the services
snapraid-daily-service-hook end
```

### RUNNING AS NON-ROOT

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

# snapraid-daily-ntfy-hook

This is a hook script that allows usage of the **ntfy** notification service here:

https://github.com/binwiederhier/ntfy

It is highly worth having a read of the documentation as the developer has put a
lot of work into it.

**ntfy** is very simple to use and very quick to set up. Once you have it setup,
to use this hook script with **snapraid-daily**, put the following into **snapraid-daily.conf**

```bash
notification_hook="/usr/bin/snapraid-daily-ntfy-hook"
export ntfy_url="https://ntfy.sh/channel_name"
```

If you are self hosting your own instance of **ntfy**, change the **ntfy_url** parameter
accordingly.

Its also a good idea to test out the notification hook on its own before using it
with **snapraid-daily**. To do that create what would be an email body by putting
some text into a file like so:

```bash
# Generate a test "email body"
echo "00:00:00 : This is a test" > test_file1.txt

# Generate a test attachment
echo "This is a sample SnapRAID command logfile showing an error" > test_file2.txt
```

Next call the hook script directly exactly how **snapraid-daily** will call it - where
the 1st argument is what would be the email subject, the 2nd is what would be the
email body, and the 3rd (if present) is the logfile of the command that generated
the error.

```bash
# Source the main config
source /etc/snapraid-daily.conf

# Test a Success Run
snapraid-daily-ntfy-hook "SnapRAID-DAILY: Success Test" test_file1.txt

# Test a Run that ends in Error
snapraid-daily-ntfy-hook "SnapRAID-DAILY: Warning Test" test_file1.txt test_file2.txt
```

# SOME NOTES ON CREATING HOOK SCRIPTS

These hook scripts are extremely simple and can be used as an example to create
ones own if desired.

Note that if you want the output of the hook script to be included in the email or
custom notification hook text, the main script makes its main logfile (which is used
for the email body) available to any hooks it calls through the use of export.

To use this in your own hook pipe the commands to **tee -a $main_logfile** like so:

```bash
echo "Hook Successful" | tee -a $main_logfile
```

# AUTHOR

Mark Finnan (mfinnan101@gmail.com)

# COPYRIGHT

Copyright (C) 2025 Mark Finnan

# SEE-ALSO

snapraid-daily(1), snapraid-daily.conf(1), sudo(1), snapraid(1), systemctl(1), systemd.unit(1)


