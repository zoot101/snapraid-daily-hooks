---
title: snapraid-apprise-hook
section: 1
header: User Manual
footer: snapraid-apprise-hook
---

# NAME

snapraid-daily-apprise-hook - Apprise Hook Script for SnapRAID-DAILY

# DESCRIPTION

This hook script takes a list of notification services defined in the **Apprise** "Url" notfication
format and sends a notification to each one of them.

Apprise is hugely versatile in that it can send notifications easily to **ntfy**, **Slack**,
**Telegram**, **Discord**, or Standard Email. Many more services are also supported. It is
very easy to use and comes **highly** recommended from the author!

To get started with **Apprise**, start with the documentation on Github below. It
doesn't mention it, but in the case of Debian (and likely most of its derivatives),
**Apprise** has been included in the official repos since Debian 12 (Bookworm).

* https://github.com/caronc/apprise

In that page there is a table provided with steps on how to enable the various different
notification services with **Apprise**, it is suggested to go through it as the documentation
is quite good.

The only other thing to mention here is that to have **Apprise** to send emails from
an email on standard providers to another email (which is what **mutt** does in the parent script),
this is the only syntax for the URL that the author has been able to get to work.

* **mailto://server.example:password@gmail.com?to=email-to-send-notifications-to@example.org**

(The **from** or **name** parameters don't seem to work if sending to another email, at least in the
authors experience). Oauth2 does not appear to be supported yet either. Hence, the author recommends
sticking with **mutt** in the default script for standard email notifications, and using **Apprise**
for everything else that isn't email.

Nonetheless, the script allows sending emails via **Apprise** if desired. See the documentation
for emails via **Apprise** here for more information:

* https://github.com/caronc/apprise/wiki/Notify\_email

To use this hook script with **SnapRAID-DAILY**, put the following in **snapraid-daily.conf**,
if already using a notification hook, change **notification_hook1** to **notification_hook2**
etc.

```bash
notification_hook1="/usr/bin/snapraid-daily-apprise-hook"
export apprise_url1="ntfys://channel_name"
export apprise_url2="tgram://Bot-ID/Chat-ID"
...
export apprise_urlN="etc"
export apprise_attach_runlog="yes"

# If apprise is installed in a non-standard location
# specify the path to its binary here. Uncomment if necessary
#export apprise_binary_path="/path/to/binary/apprise"
```

The "URLs" above should be in the format specifed on the Apprise documentation for example

* **ntfys://ntfy.sh/channel_name** (For ntfy)

Once again, the Urls are defined **apprise_url1** to **apprise_urlN**, where N is the number of services
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
any **Apprise** URLs that start with **mailto** or **mailtos** as Emails.

Here is a sample notification output that would be sent to services like
**ntfy** or **Telegram**:

```bash
=======================
Hostname: server.example.org
Host OS: Debian GNU/Linux 13 (Trixie)
SnapRAID Version: 12.4
SnapRAID-DAILY Version: 1.4.4
=======================
Initial Status: OK
Start Hook(s): Completed OK
Touch: Not Needed
Sync: Completed OK
Scrub: Completed OK
End Hook(s): Completed OK
=======================
Overall Result: Success
```

The **apprise_attach_runlog** setting shown above in the config file sample is used to
attach the default email body as an attachment to the non-email notification services like
**ntfy** or **Telegram**. This is useful to get more information from
notifications to services like **Telegram** or **ntfy** alone. Comment out or set
to "no" if not using.

The **apprise_binary_path** option is to override the path to the apprise binary
if the installation of apprise is in a non-standard location. Leave commented out
if not using.

Just like the main **SnapRAID-DAILY** script, if that exits in error, this hook
script also attaches the log generated by **SnapRAID** command that resulted in
error (status/diff/touch/sync/scrub).

## TESTING THE APPRISE HOOK

As before it's a good idea to test out the hook script on its own before pairing it
with **SnapRAID-DAILY**. To do that, follow the below procedure, after populating
**snapraid-daily.conf** with the required parameters talked about above.

Given the script is heavily dependent on the run log (email body) generated by the
main script to generate the compact notification body, it's best to use an output from
the main script to test the hook script.

To do that run the main **snapraid-daily** script and copy/paste the email sent into
a file. Alternatively, one can copy and paste the sample output from here:

* https://github.com/zoot101/snapraid-daily#sample-output

Next generate a sample text file with anything in it to masquerade as a command log
generated by **SnapRAID** in the event the main script exits with an error. Then do the
below to call the hook script directly just like the parent **snapraid-daily** script
will do.

```bash
# Generate a Text File to masquerade as the command log error
echo "Test1" > test_command_log.txt

# Source snapraid-daily.conf
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


