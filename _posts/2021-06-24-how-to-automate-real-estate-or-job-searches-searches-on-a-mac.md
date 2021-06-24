<script src="/copyCode.js"></script>
---
layout: post
title: how to automate real estate or job searches on a mac
---


<p align="center">
<img src="{{ site.baseurl }}/images/automate.jpeg" align="center">&nbsp;&nbsp;&nbsp;
</p>

recently i found myself waking up every day with computer chores: check websites and see if they had any new information for me. i wanted to free up this time so i built out an automated solution: once a day, if these sites change, send me an email. 

here is how i did it on macosx big sur 11.4

Open Terminal, and set an editor:
{% include codeHeader.html %}
```console
echo 'export EDITOR=nano' >> ~/.zshrc 
```
Install urlwatch:
```console
pip3 install urlwatch
```
find a page you want to monitor, in my case here is an example:
```console
https://deshow.com/advance-search/?operation=en-venta&type=all&subtipo=all&location=isabela&status=all&keyword=&price_range_min=0&price_range_max=3000000&bathrooms=&bedrooms=&pageid=25409
```

open google chrome developer tools and get the XPATH of where the new content will be displayed. [see how I do this here](https://www.youtube.com/watch?v=dvNDDg877cU)
<br/>
now add the details into urlwatch
```console
urlwatch --edit
name: “real estate watcher"
url: "https://deshow.com/advance-search/?operation=en-venta&type=all&subtipo=all&location=isabela&status=all&keyword=&price_range_min=0&price_range_max=3000000&bathrooms=&bedrooms=&pageid=25409"
filter:
  - xpath: /html/body/div[1]/div/div/div/div[3]
```

let’s get email notifications working using gmail:

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
[turn on gmail app passwords (requires two factor)](https://myaccount.google.com/apppasswords)
then type password here:
```console
urlwatch --smtp-login
```

run urlwatch for the first time to get current state:
```console
urlwatch
```
run it again to see if you have a good stable baseline:
```console
urlwatch
```

great! lets set up apple launch daemon:
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
apple loves plists, so let's set one up:
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
now let's add our script to launchcontrol:
```console
sudo launchctl load -w ~/Library/LaunchAgents/com.watch.plist
```
voilà! now you have a zero cost website monitor
