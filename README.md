# SnapRAID-DAILY-Hooks

Collection of hook scripts for **SnapRAID-DAILY**   
https://github.com/zoot101/snapraid-daily

These extend the functionality of **SnapRAID-DAILY** to add additional
notification methods, manage services or execute a list of commands at
the start and end.

# Table of Contents

- [Description](#description)
- [Installation and Setup](#installation-and-setup)
  - [Package Installation](#package-installation)
  - [Manual Installation](#manual-installation)
- [SnapRAID-DAILY Apprise Hook](#snapraid-daily-apprise-hook)
  - [Config File Options](#config-file-options)
  - [Compact Notification Body](#compact-notification-body)
  - [Testing the Apprise Hook](#testing-the-apprise-hook)
- [SnapRAID-DAILY Healthchecks Hook](#snapraid-daily-healthchecks-hook)
  - [Testing the Healthchecks Hook](#testing-the-healthchecks-hook)
- [SnapRAID-DAILY Ntfy Hook](#snapraid-daily-ntfy-hook)
  - [Ntfy Hook Config File Options](#ntfy-hook-config-file-options)
  - [Testing the Ntfy Hook](#testing-the-ntfy-hook)
- [SnapRAID-DAILY Service Hook](#snapraid-daily-service-hook)
  - [Testing the Service Hook](#testing-the-service-hook)
  - [Running as a Non-Root User](#running-as-a-non-root-user)
- [SnapRAID-DAILY Commands Hook](#snapraid-daily-commands-hook)
  - [Running Commands that Require Root as a Different User](#running-commands-that-require-root-as-a-different-user)
  - [Testing the Commands Hook](#testing-the-commands-hook)
- [Some Notes on Creating Hook Scripts](#some-notes-on-creating-hook-scripts)
- [Issues](#issues)

# Description

The following hook scripts are included (more will be added in the future)

These hook scripts then are intended to add additional notification functionality
to **SnapRAID-DAILY**.

* **snapraid-daily-apprise-hook**
* **snapraid-daily-healthchecks-hook**  
* **snapraid-daily-ntfy-hook**

This hook script is then intended to be used with the start and end hook function
of **SnapRAID-DAILY**, to stop a list of services while the main script is running
and restart them afterwards.

* **snapraid-daily-service-hook**
* **snapraid-daily-commands-hook**
  
# Installation and Setup

Just like with **SnapRAID-DAILY**, a package is provided for Debian and its 
derivatives here. If running a Debian based distro, it's recommended to install
the package rather than installing manually as all the dependencies necessary are
installed automatically.

## Package Installation

To install the **SnapRAID-DAILY-Hooks** debian package provided here, firstly
install the package for the main **SnapRAID-DAILY** script by downloading it from
the release page here:

* [https://github.com/zoot101/snapraid-daily/releases](https://github.com/zoot101/snapraid-daily/releases)

Then use apt to install the package like so:

```bash
# Better to install with apt instead of dpkg to install the dependencies
# automatically.
sudo apt update
sudo apt install ./snapraid-daily_1.5.1-1_amd64.deb
```

This is necessary as the main script **SnapRAID-DAILY** is a package dependency,
for the **SnapRAID-DAILY-Hooks** package. Next download the package for the hooks
from here:

* [https://github.com/zoot101/snapraid-daily-hooks/releases](https://github.com/zoot101/snapraid-daily-hooks/releases)

Then, the install of the **SnapRAID-DAILY-Hooks** package can be done like so:

```bash
# Once again - it's better to use apt so that the dependencies are
# automatically installed
sudo apt install ./snapraid-daily-hooks_0.4.0-1_amd64.deb
```

## Manual Installation

Alternatively to install manually, first download the latest source code archive from the releases page

* [https://github.com/zoot101/snapraid-daily/releases](https://github.com/zoot101/snapraid-daily/releases)

One could clone the repo with **git clone**, but since there may be some fully untested stuff added to the unreleased
versions of the hook scripts, I recommend sticking with what is on the releases page instead.


```bash
# Extract the Source Archive 
unzip snapraid-daily-hooks-0.4.0.zip       # For the Zip File
tar xvf snapraid-daily-hooks-0.4.0.tar.gz  # For the Tar File

cd snapraid-daily-hooks-0.4.0

# Install the scripts and manual entries manually
chmod +x snapraid-daily-*
sudo cp snapraid-daily-* /usr/bin/
sudo cp ./manual/*.1.gz /usr/share/man/man1/
```

Next ensure all dependencies are installed:

```bash
# On Debian based distros
sudo apt install coreutils snapraid bash curl gawk apprise

# On Fedora
sudo dnf install coreutils snapraid bash curl gawk apprise
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

The only other thing to mention here, since its not especially clear in the **Apprise** documentation
is that to have **Apprise** to send emails from an email on standard providers to another email with a
different "from" sender (which is what **mutt** does in the parent script), this is the syntax
for the **Apprise** type URL Required.

```bash
mailtos://server.example:password@gmail.com?to=email-to-send-notifications-to@example.org&from=server.example.org
```

While the author recommends sticking with **mutt** in the default script for standard email notifications (paritcularly
if you want to use its Oauth2 capability), and using **Apprise** for everything else that isn't email, the above works
fine to essentially do the same thing **mutt** does in the main script.

See the documentation for emails via **Apprise** here for more information:

* [https://github.com/caronc/apprise/wiki/Notify_email](https://github.com/caronc/apprise/wiki/Notify_email)

To use this hook script with **SnapRAID-DAILY**, put the following in **snapraid-daily.conf** - if already using
a notification hook change **notification_hook1** to **notification_hook2** etc.

```bash
# Path to Hook (Change if not installed via the Package)
notification_hook1="/usr/bin/snapraid-daily-apprise-hook"

# Define the Apprise "URLs"
export apprise_url1="ntfys://channel_name"
export apprise_url2="tgram://Bot-ID/Chat-ID"
...
export apprise_urlN="etc"

# Attach the main script runlog (email body)
export apprise_attach_runlog="yes"

# To disable the more compact runlog for non-email services
# set this option to "yes". Not recommended as it may cause
# formatting issues.
#export apprise_disable_compact_runlog="no"

# If apprise is installed in a non-standard location
# specify the path to its binary here. Uncomment if necessary
#export apprise_binary_path="/path/to/binary/apprise"
```

The "URLs" above should be in the format specifed on the Apprise documentation for example:

```bash
ntfys://ntfy.sh/channel_name # (For ntfy)
```

The Urls should be defined **apprise_url1** to **apprise_urlN**, where N is the number of services
one wants to use Apprise to send notifications to.

**Note that the use of "export" is important!**

They must be specified starting at **1** and going up to **N**.

Note that numbers should not be skipped. For example if apprise\_url1, apprise\_url2 and apprise\_url3 are
specified and subsequently apprise\_url5, apprise\_url6 and apprise\_url7 are also specified in the config file,
then the latter 3 are ignored as apprise\_url4 has not been defined. **Up to 20 URLs are supported**.

The script also creates a more compact version of the email log from the main **SnapRAID-DAILY** script
since services like **Telegram** or **ntfy** are more suited to shorter messages than standard email (see below section).

However, if the script is used to send emails via **Apprise**, the standard email log is
used as the email body, just like calls to **mutt** in the main script. This is done by regarding
any **Apprise** URLs that start with **mailto://** or **mailtos://** as Emails.

## Config File Options

The config file (**/etc/snapraid-daily.conf**) options for this hook script are outlined below:

**apprise_attach_runlog** : This is to attach the unformatted email body as
an attachment when sending notifications through **Apprise**, this is useful to get
more information from services like **Telegram** or **ntfy** which are automatically
suitied to shorter messages. Comment out and set to yes to enable, off by default.

**apprise_verbose_enable** : This is to enable the -vv option when calling **Apprise** to
enable its verbose mode. This is useful to aid in debugging if there are problems sending
notifications to certain services. Comment out and set to yes to enable, off by default.

**apprise_disable_compact_runlog** : This is to disable the compact notification body
for non-email services (see below). This is not recommended to use as certain
services like **ntfy** or **Telegram** are more suited to shorter messages, and this
can cause formatting issues or other errors. Comment out and set to yes to enable,
off by default.

**apprise_binary_path** : If **Apprise** is installed at a different location, this can
be used to override the default location in the users **PATH** variable. Comment out
and set accordingly to use. Defaults to the output of the command ``$ which apprise`` if not used.

See the sample **SnapRAID-DAILY** config provided here for an example that uses this hook script.

* [https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf](https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf)

## Compact Notification Body

As mentioned above, this hook script creates a more compact version of the email log as a notification body
for any **Apprise** Urls that do not start with **mailto://** or **mailtos://** (anything not an email),
since services like **Telegram** or **ntfy** are more suited to shorter messages than standard email.

Here is what a sample notification sent to **Telegram** looks like with the more compact notification body.

<p>
  <img src="/imgs/telegram_sample.png" height="600">
</p>

Its recommended to leave this functionality on to ensure proper formatted messages
to services like **Telegram** or **ntfy**, and if the user wants more information
to use the **apprise_attach_runlog** parameter to attach the full email body to the notification
as an attachment instead.

However, as mentioned above the above more compact log for any Apprise Urls that DO NOT start with
**mailto://** or **mailtos://** (non-email services) can be disabled through the use of the
**apprise_disable_compact_runlog** option. As mentioned before, this is **NOT** recommended as
it may cause formatting issues.
 
Just like the main **SnapRAID-DAILY** script, if that exits in error, this hook
script also attaches the log generated by **SnapRAID** command that resulted in
error (status/diff/touch/sync/scrub).

## Testing the Apprise Hook

As before it's a good idea to test out the hook script on its own before pairing it
with **SnapRAID-DAILY**. To do that, follow the below procedure after populating
**snapraid-daily.conf** with the required parameters talked about above.

Given the script is heavily dependent on the run log (email body) generated by the
main **SnapRAID-DAILY** script to generate the compact notification body, it's best to use an output from
the main script to test the hook script.

To do that run the main **snapraid-daily** script and copy/paste the email sent into
a file. Alternatively, one can copy and paste the sample output from here:

* [https://github.com/zoot101/snapraid-daily#sample-output](https://github.com/zoot101/snapraid-daily#sample-output)

Next generate a sample text file with anything in it to masquerade as a command log
generated by **SnapRAID** in the event the main script exits with an error. Then do the
below to call the hook script directly just like the parent **SnapRAID-DAILY** script
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
(**snapraid-daily.conf**). If already using a notification hook, change **notification_hook1**
to **notification_hook2** etc.

```bash
# Specify Path to Notification Hook
# (Change if not installed via the Debian Package)
notification_hook1="/usr/bin/snapraid-daily-healthchecks-hook"

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

# SnapRAID-DAILY Ntfy Hook

**ntfy** is a wonderful tool to setup up Android/iOS notifications that comes highly
recommended from the author. It is very quick to set up, the documentation is very good
and it is infintely flexible. It is also quite easy to Self-Host. 

This hook script is for use with **ntfy**. The **SnapRAID-DAILY** **Apprise** hook can
also be used with **ntfy**, and that is what the author recommends. However, this hook script
offers an alternative if one does not want to install **Apprise** and is using **ntfy**.

To get started one will need an instance of **ntfy** to use. While the script can be used with
the public ntfy server (ntfy.sh), its probably better to use the Apprise hook script instead for that.

The specifics about setting up **ntfy** are not covered here and left to the user.

There is a quick guide on how to get started quickly here:

* [https://docs.ntfy.sh](https://docs.ntfy.sh)

Publishing notifications are then covered here:

* [https://docs.ntfy.sh/publish](https://docs.ntfy.sh/publish)

There is very good documentation here on how to Self-Host it:

* [https://docs.ntfy.sh/install](https://docs.ntfy.sh/install)

The script works by attaching the Email Body as a text file attachment to notifications that
are sent. The email subject then is the notification title. Like the main script, if an error
was encountered, a 2nd notification is sent with the **SnapRAID** command logfile that caused
the error attached.

## Ntfy Hook Config File Options

The following config is required in **/etc/snapraid-daily.conf**.

```
# Ntfy Config
ntfy_url="https://ntfy.sh/channel_name"
ntfy_icon_url="https://url/to/icon.png" (Optional)
ntfy_verbose="yes" (Optional)
ntfy_priority=1 ( Optional - A number from 1 to 5 )

# If auth is configured - recommended for self-hosted setups:
ntfy_user=username
ntfy_password=password
```

The above configuration file parameters are covered below.

**ntfy_url** : Main ntfy server URL, examples:

* ntfy\_url="https://ntfy.sh/topic-name"     
* ntfy\_url="https://ntfy.example.org/topic-name"   

Required.

**ntfy_icon_url** : If one would like an icon to included with the notifications,
then one can specify a URL to an Icon File here. If for instance you already have
some sort of Webserver on your Self-Hosted setup, this can be added easily. Example:

* ntfy\_icon\_url="https://example.org/path/to/icon.png"

This is optional and can be omitted or commented out if not using.

**ntfy_verbose** : This is useful to have the json data received from the **ntfy** server
printed for debugging. Set to "yes" to enable. Comment out or set to "no" to disable.

This is also optional.

**ntfy_priority** : **ntfy** allows one to specify different notification priorities to vary the duration of the
vibration bursts among other things. It is covered in the documentation here:

* [https://docs.ntfy.sh/publish](https://docs.ntfy.sh/publish) 

It can be specified as a number from 1 to 5 here. Comment out or omit to disable and
assume the default of 3.

**ntfy_user & ntfy_password** : The default configuration of **ntfy** allows anyone to
publish anything to any topic. This is probably not desirable for a Self-Hosted setup
particularly if it accessible from the internet. These parameters
allow one to specify a username and password to use for authorisation. Make sure that the server is setup
with https if using.

Omit or leave blank to disable and allow anyone to publish anything to any topic.

For an entirely local setup, these can be safely omitted to assume the default setup.

## Testing the Ntfy Hook

Finally it is a good idea to test the script out on its own before using it with **snapraid-daily**
directly. One will need a sample email output to test the script.

Either let the main script complete and copy & paste the email into a file, or copy the sample
output from the Github page here:

* [https://github.com/zoot101/snapraid-daily-hooks](https://github.com/zoot101/snapraid-daily-hooks)

Then, do the below: 

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test the start commands
snapraid-daily-ntfy-hook "SnapRAID-DAILY: All OK" "/path/to/sample/email/body.txt"

# To test out the error scenario with a command log
snapraid-daily-ntfy-hook "SnapRAID-DAILY: All OK" "/path/to/sample/email/body.txt" "/path/to/sample/command/log.txt"
```

# SnapRAID-DAILY-Service-Hook

This script stops a list of services using **systemd** when the main script
starts, and restarts them when the main script finishes. It is intended to be used
with the start and end hook feature of **SnapRAID-DAILY**.

To use it, specify the following in the main script configuration file (**snapraid-daily.conf**),
if already using a start and end hook, change **start_hook1** to **start_hook2** and **end_hook1**
to **end_hook2** etc.

```bash
# Specify path to hook script for start_hook and end_hook
# parameters - change if needed
start_hook1="/usr/bin/snapraid-daily-service-hook"
end_hook1="/usr/bin/snapraid-daily-service-hook"

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

When the script is called with the "start" argument, which is what **SnapRAID-DAILY** does at the
start, if any errors are encountered while attempting to stop any one of the services, the script will
exit immediately and hand control back to **SnapRAID-DAILY** which will exit and sent the user
an email or call the notification hook(s).

On the other hand, if errors are encountered while attempting to restart any of the services listed in
the main config file, the script will continue to the end and attempt to restart all services before
handing back control to **SnapRAID-DAILY**, which in turn will continue to the end and notify
the user accordingly.

The thinking here is that it is more important to attempt to ***re-start*** all services defined ni
the config file than attempt to stop all services listed.

See the sample **SnapRAID-DAILY** config provided here for an example that uses this hook
script.

* [https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf](https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf)

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

If one is running the main **SnapRAID-DAILY** script as root this section does not apply, as
typically root is required to use the **systemctl** command to stop/start services.

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

# SnapRAID-DAILY Commands Hook

This script executes a list of commands using **bash** when the main **SnapRAID-DAILY** script
starts or finishes (either in success or warning/error). It offers an alternative of
writing a very simple script to accomplish the same.

A common potential use-case may be to mount the Parity Disk(s) when **SnapRAID-DAILY** starts
and umount them when **SnapRAID-DAILY** completes. Another one may be to run a number of backup
jobs with rsync at the start.

To use it, specify the following in the main script configuration file (**snapraid-daily.conf**),
if already using a start/end hook change **start_hook1** to **start_hook2** or **end_hook1**
to **end_hook2** etc.

```bash
# Specify path to script
start_hook1="/usr/bin/snapraid-daily-commands-hook"
end_hook1="/usr/bin/snapraid-daily-commands-hook"

# Specify Start Commands (Up to 5 are Supported)
export start_command1="rsync -av --delete /path1/ /path2/"
export start_command2="cp /path/to/pictures1/ /path/to/pictures2"
...
...
export start_commandN="whatever"

# Specify End Commands (Up to 5 are Supported)
export end_command1="rsync -av --delete /path3/ /path4/"
export end_command2="cp /path/to/pictures5/ /path/to/pictures6"
...
...
export end_commandN="whatever"
```

(Where N is the number of commands one wishes to call at the start or end of the run
of **SnapRAID-DAILY**)

The start and end hook parameter are set to point to the hook script directly. A list of
commands is also required to pass into the script to execute upon start and end. **Note that**
**the use of "export" is important!**

The commands must be specified as **start_command1**= , **start_command2**= .... **start_commandN**= .
That is starting at **1** and going up to **N**, where **N** is the number of commands. It would be much
easier to specify an array here, but it isn't possible to export arrays in bash sadly.

Note that numbers should not be skipped. For example if **start_command1**, **start_command2** and
**start_command3** are given and subsequently **start_command5** is specified in the config file,
then the latter command is ignored. **Up to 5 Commands are supported**.

(The above is also true with the **end_hook1** to **end_hook5** parameters in the config file)

The above commands can be anything that is called from the command line, they can be standard
commands using standard installed programs, or can be seperate scripts on their own. Note that they
should be enclosed in "" in the config file. Example:

```bash
start_command1="sudo mount /mnt/parity_disk1")
```

If any special characters are required they should be escaped using **\"\\"**. Example:

```bash
start_command1="for f in /path1/;do rm \$f;done"
```

If any of the start commands specified end in an error condition, the script will return
an error to the main **SnapRAID-DAILY** script which will exit and notify the user
accordingly.

If any of the end commands end in an error condition, the script will continue to the
end to be consistent with the main **SnapRAID-DAILY** script.

See the sample **SnapRAID-DAILY** config provided here for an example that uses this hook
script.

* [https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf](https://github.com/zoot101/snapraid-daily/blob/main/docs/examples/snapraid-daily.conf)

## Running Commands that Require Root as a Different User

If one is running the main script as **root** this section does not apply.

In the event that one is running the main script as a user other than root and wants to use
this hook to run a number of commands that require root - one can use **sudo** by just specifying
the commands in the config file like so:

```bash
export start_command1="sudo mount /mnt/parity_disk1"   
export end_command1="sudo umount /mnt/parity_disk1"   
```

The above example will mount a Parity drive upon the start and unmount it after **SnapRAID-DAILY**
completes. This assumes that no content files are on the parity disk, and also that it is set up
appropriately in **/etc/fstab** (Mounted at /mnt/parity_disk1 in the above example)

Taking the above example - to allow usage of the **mount** command for your user without a password
create a file in **/etc/sudoers.d** like so. This should also work for any other command that requires
root.

```bash
# As root do
visudo /etc/sudoers.d/username

# Paste in the following (change your username), save and close
username ALL=(root) NOPASSWD:/usr/bin/mount
```

# Testing the Commands Hook

Finally it is a good idea to test the script out on its own before using it with **snapraid-daily**
directly. Do that like so (as root) by calling the hook script exactly how **snapraid-daily** will
call it, after populating **snapraid-daily.conf** with the above information:

```bash
# Source the main config
source /etc/snapraid-daily.conf

# To test the start commands
snapraid-daily-commands-hook start

# To test the end commands
snapraid-daily-commands-hook end
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

# Issues

Just like with the main script - bug reports here on Github are welcome - don't hestitate if you find something wrong.

* [https://github.com/zoot101/snapraid-daily-hooks/issues](https://github.com/zoot101/snapraid-daily-hooks/issues)

