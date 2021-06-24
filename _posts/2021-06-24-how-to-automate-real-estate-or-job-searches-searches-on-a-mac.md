---
layout: post
title: how to automate real estate or job searches searches on a mac
---


<p align="center">
<img src="{{ site.baseurl }}/images/automate.jpeg" align="center">&nbsp;&nbsp;&nbsp;
</p>

recently i found myself waking up every day with computer chores: check the same websites every day and see if they had any new information for me. I wanted to free up this time so built out this simple automation script: once a day, if these sites change, send me an email. 

it turns out there is a great tool written to do this: urlwatch

i wanted it to run on my laptop, even when it might have been hibernating, so I opted to use Apple’s Launch Daemon.

Here is how you can do something similar:

Open Terminal, and set an editor:
```console
echo 'export EDITOR=nano' >> ~/.zshrc 
```
Install urlwatch:

```console
pip3 install urlwatch
```
Find a page you want to monitor, in my case here is an example:
```console
"https://deshow.com/advance-search/?operation=en-venta&type=all&subtipo=all&location=isabela&status=all&keyword=&price_range_min=0&price_range_max=3000000&bathrooms=&bedrooms=&pageid=25409
```

Open google chrome developer tools and get the XPATH of where the new content will be displayed. See how I do this here:
[https://www.youtube.com/watch?v=dvNDDg877cU]
Now add the details into urlwatch
```console
urlwatch --edit
name: “real estate watcher"
url: "https://deshow.com/advance-search/?operation=en-venta&type=all&subtipo=all&location=isabela&status=all&keyword=&price_range_min=0&price_range_max=3000000&bathrooms=&bedrooms=&pageid=25409"
filter:
  - xpath: /html/body/div[1]/div/div/div/div[3]
```

Let’s get the email notification settings set up using gmail:

```console
urlwatch --edit-config
report:
  discord:
    embed: false
    enabled: false
    max_message_length: 2000
    subject: '{count} changes: {jobs}'
    webhook_url: ''
  email:
    enabled: true
    from: 'YOUREMAIL'
    html: false
    method: smtp
    sendmail:
      path: sendmail
    smtp:
      keyring: true
      host: smtp.gmail.com
      port: 587
      starttls: true
      user: 'YOUREMAIL'
    subject: '{count} changes: {jobs}'
    to: 'YOUREMAIL'
```
Turn on your gmail app password (requires two factor): [https://myaccount.google.com/apppasswords]
Then type password here:
```console
urlwatch --smtp-login
```

Run urlwatch for the first time to get current state:
```console
urlwatch
```
Run it again to see if you have a good stable baseline:
```console
urlwatch
```

Great! Lets set up Apple Launch Daemon:
```console
sudo su
nano /Library/Scripts/watch.sh
```
script content:
```console
#!/bin/bash
	/usr/local/bin/urlwatch
```
exit root:
```console
exit
```
apple loves plists, so let's set it up:
```console
nano ~/Library/LaunchAgents/com.watch.plist 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.watch</string>
    <key>Program</key>
    <string>/Library/Scripts/watch.sh</string>
    <key>StandardOutPath</key>
    <string>/tmp/watch.sh.stdout</string>
    <key>StandardErrorPath</key>
    <string>/tmp/watch.sh.stderr</string>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>12</integer>
        <key>Minute</key>
        <integer>10</integer>
    </dict>
</dict>
</plist>
```
Now let's add our script to LaunchControl:
```console
sudo launchctl load -w ~/Library/LaunchAgents/com.watch.plist
```
