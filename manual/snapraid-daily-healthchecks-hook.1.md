---
title: snapraid-daily-healthchecks-hook
section: 1
header: User Manual
footer: snapraid-daily-healthchecks-hook
---

# NAME

snapraid-daily-healthchecks-hook - Hook script to allow use of Healthchecks.io
with snapraid-daily.

# DESCRIPTION

This is a hook script for use with **Healthchecks.io** here:

* https://healthchecks.io

It is a nice service that allows one to monitor up to 20 scheduled tasks for free,
and if one of those tasks is found to have a problem, a wide varity of notification
services can be configured on **healthchecks.io** like **Discord**, **Telegram**,
Standard Email etc.

To use this hook, the following is required in the snapraid-daily config file
(**snapraid-daily.conf**), if already using a notification hook change
**notification_hook1** to **notification_hook2** etc.

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

## TESTING THE HEALTHCHECKS HOOK

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


