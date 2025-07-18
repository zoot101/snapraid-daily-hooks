# SnapRAID-DAILY-Hooks

Collection of hook scripts for **SnapRAID-DAILY**   
https://github.com/zoot101/snapraid-daily

These extend the functionality of **SnapRAID-DAILY** to add additional
notification methods or to stop a list of services while the main script
is running and restart them afterwards.

# Table of Contents

- [Description](#description)
- [Installation and Setup](#installation-and-setup)
- [SnapRAID-DAILY Apprise Hook](#snapraid-daily-apprise-hook)
  - [Testing the Apprise Hook](#testing-the-apprise-hook)
- [SnapRAID-DAILY Healthchecks Hook](#snapraid-daily-healthchecks-hook)
  - [Testing the Healthchecks Hook](#testing-the-healthchecks-hook)
- [SnapRAID-DAILY Service Hook](#snapraid-daily-service-hook)
  - [Testing the Service Hook](#testing-the-service-hook)
  - [Running as a Non-Root User](#running-as-a-non-root-user)
- [Some Notes on Creating Hook Scripts](#some-notes-on-creating-hook-scripts)

# Description

The following hook scripts are included (more will be added in the future)

These hook scripts then are intended to add additional notification functionality
to **SnapRAID-DAILY**.

* **snapraid-daily-apprise-hook**
* **snapraid-daily-healthchecks-hook**  

This hook script is then intended to be used with the start and end hook function
of **SnapRAID-DAILY**, to stop a list of services while the main script is running
and restart them afterwards.

* **snapraid-daily-service-hook**
  
# Installation and Setup

Just like with **SnapRAID-DAILY**, a package is provided for Debian and its 
derivatives here. If running a Debian based distro, it's recommended to install
the package rather than installing manually as all the dependencies necessary are
installed automatically.

To install the **SnapRAID-DAILY-Hooks** debian package provided here, firstly
install the package for the main **SnapRAID-DAILY** script by downloading it from
the release page here:

* [https://github.com/zoot101/snapraid-daily/releases](https://github.com/zoot101/snapraid-daily/releases)

Then use apt to install the package like so:

```bash
# Better to install with apt instead of dpkg to install the dependencies
# automatically.
sudo apt update
sudo apt install ./snapraid-daily_1.4.1-1_amd64.deb
```

This is necessary as the main script **SnapRAID-DAILY** is a package dependency,
for the **SnapRAID-DAILY-Hooks** package. Next download the package for the hooks
from here:

* [https://github.com/zoot101/snapraid-daily-hooks/releases](https://github.com/zoot101/snapraid-daily-hooks/releases)

Then, the install of the **SnapRAID-DAILY-Hooks** package can be done like so:

```bash
# Once again - it's better to use apt so that the dependencies are
# automatically installed
sudo apt install ./snapraid-daily-hooks_0.3.0-1_amd64.deb
```

Alternatively to install manually, do the following:

```bash
# (On Debian based distros)
sudo apt update
sudo apt install git

# (On Fedora)
sudo dnf update
sudo dnf install git-core # (On Fedora)

# Clone the Git repo
git clone https://github.com/zoot101/snapraid-daily-hooks
cd snapraid-daily-hooks

# Install the scripts and manual entry manually
chmod +x snapraid-daily-*
sudo cp snapraid-daily-* /usr/bin/
sudo cp ./manual/snapraid-daily-hooks.1.gz /usr/share/man/man1/
```

Next ensure all dependencies are installed:

```bash
# On Debian based distros
sudo apt install coreutils snapraid bash jq curl gawk apprise

# On Fedora
sudo dnf install coreutils snapraid bash jq curl gawk apprise
```

# SnapRAID-DAILY Apprise Hook

This hook script takes a list of notification services defined in the **Apprise** "Url" notfication
format and sends a notification to each one of them.

Apprise is hugely versatile in that it can send notifications easily to **ntfy**, **Slack**,
**Telegram**, **Discord**, or Standard Email. Many more services are also supported. It is
very easy to use and comes **highly** recommended from the author!

To get started with **Apprise**, start with the documentation on Github below. It
doesn't mention it, but in the case of Debian (and likely most of its derivatives),
**Apprise** has been included in the official repos since Debian 12 (Bookworm).

* [https://github.com/caronc/apprise](https://github.com/caronc/apprise)

In that page there is a table provided with steps on how to enable the various different
notification services with **Apprise**, it is suggested to go through it as the documentation
is quite good.

The only other thing to mention here is that to have **Apprise** to send emails from
an email on standard providers to another email (which is what **mutt** does in the parent script),
this is the only syntax for the URL that the author has been able to get to work.

```bash
mailto://server.example:password@gmail.com?to=email-to-send-notifications-to@example.org
```

(The **from** or **name** parameters don't seem to work if sending to another email, at least in the
authors experience). Oauth2 does not appear to be supported yet either. Hence, the author recommends
sticking with **mutt** in the default script for standard email notifications, and using **Apprise**
for everything else that isn't email.

Nonetheless, the script allows sending emails via **Apprise** if desired. See the documentation
for emails via **Apprise** here for more information:

* [https://github.com/caronc/apprise/wiki/Notify_email](https://github.com/caronc/apprise/wiki/Notify_email)

To use this hook script with **SnapRAID-DAILY**, put the following in **snapraid-daily.conf**:

```bash
# Path to Hook (Change if not installed via the Package)
notification_hook="/usr/bin/snapraid-daily-apprise-hook"

# Define the Apprise "URLs"
export apprise_url1="ntfys://channel_name"
export apprise_url2="tgram://Bot-ID/Chat-ID"
...
export apprise_urlN="etc"

# Attach the main script runlog (email body)
export apprise_attach_runlog="yes"
```

The "URLs" above should be in the format specifed on the Apprise documentation for example

```bash
ntfys://ntfy.sh/channel_name # (For ntfy)
```

The Urls should be defined **apprise_url1** to **apprise_urlN**, where N is the number of services
one wants to use Apprise to send notifications to.

**Note that the use of "export" is important!**

They must be specified starting at **1** and going up to **N**. It would be much easier to specify
an array here, but it isn't possible to export arrays in bash sadly, so it would require a
seperate config file for the hook script which is messy in the opinion of the author.

Note that numbers should not be skipped. For example if apprise\_url1, apprise\_url2 and apprise\_url3 are
specified and subsequently apprise\_url5, apprise\_url6 and apprise\_url7 are also specified in the config file,
then the latter 3 are ignored as apprise\_url4 has not been defined. **Up to 20 URLs are supported**.

The hook script also creates a more compact version of the email log since services like
**Telegram** or **ntfy** are more suited to shorter messages than standard email.

Note that if the script is used to send emails via **Apprise**, the standard email log is
used as the email body, just like calls to **mutt** in the main script. This is done by regarding
any **Apprise** URLs that start with **mailto://** or **mailtos://** as Emails.

Here is a sample notification output that would be sent to services like
**ntfy** or **Telegram**:

```bash
SnapRAID-DAILY: All OK
=======================
Hostname: server.example.org
Host OS: Debian GNU/Linux 13 (Trixie)
SnapRAID Version: 12.4
=======================
Initial Status: OK
Start Hook: Completed OK
Touch: Not Needed
Sync: Completed OK
Scrub: Completed OK
End Hook: Completed OK
```

The **apprise_attach_runlog** setting shown above in the config file sample is used to
attach the default email body as an attachment to the non-email notification services like
**ntfy** or **Telegram**.

This is useful to get more information from notifications to services like **Telegram**
or **ntfy** alone, and allows them to act as a true substitue for standard emails if desired.
Comment out or set to "no" if not using.

Just like the main **SnapRAID-DAILY** script, if that exits in error, this hook
script also attaches the log generated by **SnapRAID** command that resulted in
error (status/diff/touch/sync/scrub).

See the sample **SnapRAID-DAILY** config provided here for an example that uses this hook
script.

* [https://github.com/zoot101/snapraid-daily/blob/main/docs/sample-config/snapraid-daily.conf](https://github.com/zoot101/snapraid-daily/blob/main/docs/sample-config/snapraid-daily.conf)

## Testing the Apprise Hook

As before it's a good idea to test out the hook script on its own before pairing it
with **SnapRAID-DAILY**. To do that, follow the below procedure, after populating
**snapraid-daily.conf** with the required parameters talked about above.

Given the script is heavily dependent on the run log (email body) generated by the
main script to generate the compact notification body, it's best to use an output from
the main script to test the hook script.

To do that run the main **snapraid-daily** script and copy/paste the email sent into
a file. Alternatively, one can copy and paste the sample output from here:

* [https://github.com/zoot101/snapraid-daily#sample-output](https://github.com/zoot101/snapraid-daily#sample-output)

Next generate a sample text file with anything in it to masquerade as a command log
generated by **SnapRAID** in the event the main script exits with an error. Then do the
below to call the hook script directly just like the parent **snapraid-daily** script
will do.

```bash
# Generate a Text File to masquerade as the command log error
echo "Test1" > test_command_log.txt

# Source snapraid-daily.conf directly to load variables
source /etc/snapraid-daily.conf

# Call the Hook script to test a Successfull Run
# (sample_email_body.txt is the output from the main script discussed above)
snapraid-daily-apprise hook "Test Subject" "sample_email_body.txt"

# Call the Hook script to test a Run ending with a Warning/Error
snapraid-daily-apprise-hook "Test Subject" "sample_email_body" "test_command_log.txt"
```

If the above commands correctly send notifications to your desired notification
services, then the hook script is configured correctly and ready for use
with **SnapRAID-DAILY**.

# SnapRAID-DAILY Healthchecks Hook

This is a hook script for use with **Healthchecks.io** here:

* [https://healthchecks.io](#https://healthchecks.io)

It is a nice service that allows one to monitor up to 20 scheduled tasks for free,
and if one of those tasks is found to have a problem, a wide varity of notification
services can be configured on **healthchecks.io** like **Discord**, **Telegram**,
Standard Email etc.

To use this hook, the following is required in the snapraid-daily config file
(**snapraid-daily.conf**)

```bash
# Specify Path to Notification Hook
# (Change if not installed via the Debian Package)
notification_hook="/usr/bin/snapraid-daily-healthchecks-hook"

# Healthchecks.io URLs
healthchecks_url="https://hc-ping.com/123...456...abc"
healthchecks_sync_url="https://hc-ping.com/123abzz...123"
healthchecks_scrub_url="https://hc-ping.com/78990...frtg"
```

The 3 URL settings are explained below.

The **healthchecks_url** is intended to be used if one is running the main
**snapraid-daily** script in the default configuration - where a sync operation
and scrub operation are carried out each time the main **snapraid-daily** script
is called. (Neither the **-s|--sync-only** or **-c|--scrub-only** arguments are
being used).

This URL is contacted each time **snapraid-daily** finishes, and the **healthchecks.io**
server is explicitly notified of a successful run or a run ending in error/warning
after the main **snapraid-daily** script complets.

The **healthchecks_sync_url** and **healthchecks_scrub_url** URLs are intended to
account for the case whereby one is seperating the sync and scrub operations of the
main **snapraid-daily** script, that is one is using the **-s|--sync-only** or
**-c|--scrub-only** arguments when the parent **snapraid-daily** script is ran.

For sync operations the **healthchecks_sync_url** is contacted to specify
either success or error/warning if the main **snapraid-daily** script is called with
the **-s|--sync-only** argument.

For scrub operations the **healthchecks_scrub_url** is contacted to specify
either success or error/warning if the main **snapraid-daily** script is called with
the **-c|--scrub-only** argument.

These are both optional and can be omitted from **snapraid-daily.conf** if one wants
to just the **healthchecks_url** setting only regardless of the input arguments to the
parent **snapraid-daily** script. Alternatively just one of them can be used also.

## Testing the Healthchecks Hook

It is a good idea to test out the hook script on its own before using it with
**snapraid-daily** directly. To do that, follow the below procedure.

Create a sample email body file (body1.txt) that contains the following lines:

* **Run-Sync: YES**   
* **Run-Scrub: YES**   

These are part of the intial greeting **snapraid-daily** prints out and are used
as a basis to determine what the main script was called with.

To simulate a default run, use the above file contents. To simulate a sync operation only or
a scrub operation only, change the corresponding "YES" above to "NO" accordingly.

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

One can then inspect the **healthchecks.io** WebUI to see if the expected result is
produced. Note the "body3.txt" argument isn't important for the purposes of testing
above, it can be anything, it doesn't even have to be a file.

# SnapRAID-DAILY-Service-Hook

This script stops a list of services using **systemd** when the main script
starts, and restarts them when the main script finishes. It is intended to be used
with the start and end hook feature of **SnapRAID-DAILY**.

To use it, specify the following in the main script configuration file (**snapraid-daily.conf**)

```bash
# Specify path to hook script for start_hook and end_hook
# parameters - change if needed
start_hook="/usr/bin/snapraid-daily-service-hook"
end_hook="/usr/bin/snapraid-daily-service-hook"

# Specify the serivces to be stopped and then later
# re-started
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

Just like above, the services must be specified as service1= , service2= .... serviceN= .
Starting at 1 and going up to N, where N is the number of services to stop/start. Note also
NOT to include the ".service" at the end when defining the services in the config file as this
will cause the script to exit with an error. It would be much easier to specify an array here,
but it isn't possible to export arrays in bash sadly.

Note that numbers should not be skipped. For example if service1, service2 and service3 are
given and subsequently service5, service6 and service7 are specified in the config file,
then the later 3 are ignored. **Up to 20 Services are supported**.

See the sample **SnapRAID-DAILY** config provided here for an example that uses this hook
script.

* [https://github.com/zoot101/snapraid-daily/blob/main/docs/sample-config/snapraid-daily.conf](https://github.com/zoot101/snapraid-daily/blob/main/docs/sample-config/snapraid-daily.conf)

## Testing the Service Hook

Finally it is a good idea to test the script out on its own before using it with **SnapRAID-DAILY**
directly. Do that like so (as root) by calling the hook script exactly how **SnapRAID-DAILY** will
call it:

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test stopping the services
# Call the hook with a "start" argument
snapraid-daily-service-hook start

# To test re-starting the services
# Call the hook with an "end" argument
snapraid-daily-service-hook end
```

## Running as a Non-Root User

Typically root is required to use the **systemctl** command to stop/start services.

However, if one does not run the main **SnapRAID-DAILY** script as root, the hook script is also
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
main **SnapRAID-DAILY** script.

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test stopping the services
snapraid-daily-service-hook start

# To test re-starting the services
snapraid-daily-service-hook end
```

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

