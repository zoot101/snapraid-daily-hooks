# SnapRAID-DAILY-Hooks

Collection of hook scripts for **SnapRAID-DAILY**   
https://github.com/zoot101/snapraid-daily

These extend the functionality of **snapraid-daily** to add additional
notification methods or to stop a list of services while the main script
is running and restart them afterwards.

# Table of Contents

- [Description](#description)
- [Installation and Setup](#installation-and-setup)
- [SnapRAID-DAILY-Service-Hook](#snapraid-daily-service-hook)
  - [Running as a Non-Root User](#running-as-a-non-root-user)
- [SnapRAID-Daily-ntfy-Hook](#snapraid-daily-ntfy-hook)
  - [ntfy Hook Test](#ntfy-hook-test)
- [SnapRAID-Daily-Healthchecks-Hook](#snapraid-daily-healthchecks-hook)
  - [Healthchecks URLs](#healthchecks-urls)
  - [Healthchecks Hook Test](#healthchecks-hook-test)
- [Some Notes on Creating Hook Scripts](#some-notes-on-creating-hook-scripts)

# Description

The following hook scripts are included (more will be added in the future)

These hook scripts are intended to be used with the start and end hook function
of **snapraid-daily**.

* **snapraid-daily-service-hook**

These hook scripts then are intended to add additional notification functionality
to **snapraid-daily**.

* **snapraid-daily-ntfy-hook**   

# Installation and Setup
Just like with **snapraid-daily**, a package is provided for Debian and its 
derivatives here.

To install the package download it from the releases page. Note that the main
script **snapraid-daily** is a package dependency, so install that package first
from here:   
https://github.com/zoot101/snapraid-daily

Then, the install of the package can be done like so:

```bash
# Install main snapraid-daily
sudo apt install ./snapraid-daily_1.4.1-1_amd64.deb

# Then install the hooks package
sudo apt install ./snapraid-daily-hooks_0.1.1-1_amd64.deb
```

Note that it's much better to use **apt** rather than **dpkg** so the dependencies will be
automatically installed.

Alternatively to install manually, do the following:

```bash
sudo apt install git # (On Debian based distros)
sudo dnf install git-core # (On Fedora)

git clone https://github.com/zoot101/snapraid-daily-hooks
cd snapraid-daily-hooks

chmod +x snapraid-daily-*

sudo cp snapraid-daily-* /usr/bin/
sudo cp ./manual/snapraid-daily-hooks.1.gz /usr/share/man/man1/
```

Next ensure all dependencies are installed:

```bash
# On Debian based distros
sudo apt install coreutils snapraid bash jq curl gawk

# On Fedora
sudo dnf install coreutils snapraid bash jq curl gawk
```

# SnapRAID-DAILY-Service-Hook

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

## Running as a Non-Root User

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

# SnapRAID-DAILY-ntfy-Hook

This is a hook script that allows usage of the **ntfy** notification service here:

https://github.com/binwiederhier/ntfy

It is highly worth having a read of the documentation as the developer has put a
lot of work into it.

**ntfy** is very simple to use and very quick to set up, it is a really good tool that
is very easy to get notifications on your phone, it is also very easy to self host. It
comes highly recommended from the author!

Once you have it setup from following the above documentation, to use this hook script
with **snapraid-daily**, put the following into **snapraid-daily.conf**

```bash
notification_hook="/usr/bin/snapraid-daily-ntfy-hook"
export ntfy_url="https://ntfy.sh/channel_name"
export ntfy_attach_run_log="no"
```

If you are self hosting your own instance of **ntfy**, change the **ntfy_url** parameter
accordingly.

Because push notifications are geared more towards short messages, the full email
log is not used as the body of the notification message. Instead the much shorter
email subject ex. "SnapRAID-DAILY: All OK" is used as the text body.

If one wants to attach the full run-log (email body of **snapraid-daily**) as a
seperate notification set the **ntfy_attach_run_log** parameter to **yes** in
**snapraid-daily.conf** like above. Comment out or omit if the email subject
is enough.

If errors are encountered during the script, just as with the email notifications
the logfile from the SnapRAID command that generated the error is attached. This is
done by sending a seperate notification again and attaching the command error log.
This means that up to 3 notifications are sent depending on the circumstances and
input options being used.

## ntfy Hook Test

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
# Source the main config (to load above variables)
source /etc/snapraid-daily.conf

# Test a Success Run
snapraid-daily-ntfy-hook "SnapRAID-DAILY: Success Test" test_file1.txt

# Test a Run that ends in Error
snapraid-daily-ntfy-hook "SnapRAID-DAILY: Warning Test" test_file1.txt test_file2.txt
```

# SnapRAID-DAILY-Healthchecks-Hook

This is a hook script for use with Healthchecks.io here:

https://healthchecks.io

It is a nice service that allows one to monitor up to 20 scheduled tasks for free,
and if one of those tasks is found to have a problem, a wide varity of notification
services can be configured on healthchecks.io like Discord, Telegram, Standard
Email etc.

To use this hook, the following is required in the snapraid-daily config file
(**snapraid-daily.conf**)

```bash
# Specify Path to Notification Hook
notification_hook="/usr/bin/snapraid-daily-healthchecks-hook"

# Healthchecks.io URLs
healthchecks_url="https://hc-ping.com/123...456...abc"
healthchecks_sync_url="https://hc-ping.com/123abzz...123"
healthchecks_scrub_url="https://hc-ping.com/78990...frtg"
```

## Healthchecks URLs

The 3 URL settings are explained below.

### healthchecks\_url

This is intended if one is running the main **snapraid-daily** script in the
default configuration where a sync operation and scrub operation are carried
out each time the main **snapraid-daily** script is called. (**-s|--sync-only**
or **-c|--scrub-only** arguments are not being used).

This URL is contacted each time **snapraid-daily** finishes, and the healthchecks.io
server is explicitly notified of a successful run or a run ending in error/warning
after the main **snapraid-daily** script complets.

### healthchecks\_sync\_url & healthchecks\_scrub\_url

These URLs are intended to account for the case whereby one is seperating the
sync and scrub operations of the main **snapraid-daily** script.

For sync operations the **healthchecks_sync_url** is contacted to specify
either success or error/warning if the main **snapraid-daily** script is called with
the **-s|--sync-only** argument.

For scrub operations the **healthchecks_scrub_url** is contacted to specify
either success or error/warning if the main **snapraid-daily** scrilpt is called with
the **-c|--scrub-only** argument.

These are both optional and can be omitted from **snapraid-daily.conf** if one wants
to just the **healthchecks_url** setting only. Alternatively just one of them can
be used also.

## Healthchecks Hook Test

It is a good idea to test out the hook script on its own before using it with
**snapraid-daily** directly. To do that, follow the below procedure.

Create a sample email body file (body1.txt) that contains the following lines:

* **Run-Sync: YES**   
* **Run-Scrub: YES**   

These are part of the intial greeting **snapraid-daily** prints out and are used
as a basis to determine what the main script was called with.

To simulate a default run, use the above file contents. To simulate a sync operation only or
a scrub operation only, change the "YES" to "NO" accordingly.

Then populate **snapraid-daily.conf** with the path to the hook script and the
desired healthchecks.io URLs, and follow the procedure below.

Call the hook script directly exactly how **snapraid-daily** will call it - where
the 1st argument is what would be the email subject, the 2nd is what would be the
email body, and the 3rd (if present) is the logfile of the command that generated
the error.

```bash
# Source main snapraid-daily.conf to load variables
source snapraid-daily.conf

# Test a Success Run
/usr/bin/snarpaid-daily-healthchecks-hook "SnapRAID-DAILY: All OK" "body1.txt"

# Test an Error/Warning Run
/usr/bin/snapraid-daily-healthchecks-hook "SnapRAID-DAILY: Sync Warning(s)" "body1.txt" "body3.txt"
```

One can then inspect the healthchecks.io WebUI to see if the expected result is
produced. Note the "body3.txt" argument isn't important above, it can be anything.

# Some Notes on Creating Hook Scripts

These hook scripts are extremely simple and can be used as an example to create
ones own if desired.

Note that if you want the output of the hook script to be included in the email or
custom notification hook text, the main script makes its main logfile (which is used
for the email body) available to any hooks it calls through the use of export.

To use this in your own hook pipe the commands to **tee -a $main_logfile** like so:

```bash
echo "Hook Successful" | tee -a $main_logfile
```

