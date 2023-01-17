---
title: Setting Up Code Server On A Raspberry Pi
description: meta description
image: images/post/05.jpg
date: 2022-01-16T19:00:00+00:00
author: Antoine Gosset
tags:
- RaspberryPi
categories:
- RaspberryPi

---
## Setting up the Raspbian image

To avoid any compatibility issues, we are going to install the 64-BIT version of Raspbian so most of ARM64 binaries can be used

1. Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) (v1.7.3 at the time of writing)
2. Open Raspberry Pi Imager
3. Select the operating system `RASPBERRY PI OS LITE (64-BIT)`
4. Select the storage used (USB or SD card)
5. Click the `config` icon to access the advanced options

* Enable hostname and add a name (`codepi` in our case)
* Enable SSH and use password authentication
* Enable username and password (`codepi`, ****)
* Enable wireless LAN and fill in information about SSID and password for your network
* Enable local settings (`America/Los_angeles` in our case)
* Click the save button

1. Click the `WRITE` button and let the process finish
2. Plug the SD card or USB stick into the pi ad let it boot. It may take \~1-2 min as multiple reboots are required during the first setup

## Installing Code Server

TIP: if you dont know the IP address of your raspberrry pi, open a terminal and try to ping it by using its hostname

```bash
ping codepi.local
```

If the raspberry pi is online, you will see in the terminal its IP address

```bash
PING codepi.local (192.168.0.106): 56 data bytes
  64 bytes from 192.168.0.106: icmp_seq=0 ttl=64 time=8.094 ms
  64 bytes from 192.168.0.106: icmp_seq=1 ttl=64 time=9.508 ms
  64 bytes from 192.168.0.106: icmp_seq=2 ttl=64 time=16.727 ms
```

Let's connect to our freshly installed raspberry pi through ssh:

```bash
ssh codepi@192.168.0.106
```

If this is the first time connecting to it, host key will not be known and you will be asked if you want to connect to it.

Type `yes` followed by the `RETURN` key

Congratulation! You have now a functional raspberry pi ready to be configured :)

[code-server](https://github.com/cdr/code-server) is free, open-source, and can be run on any machine with few limitations, including the Raspberry Pi.

Following the recommendations from their [README](https://github.com/cdr/code-server#getting-started), I'll be using the install script, which automates most of the process.

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

Be patient, the installation took around 15 minutes in my RPi Model 3B+. Once completed, you should have a similar output to:

```bash
Debian GNU/Linux 11 (bullseye)
Installing v4.9.1 of the arm64 deb package from GitHub.

+ mkdir -p ~/.cache/code-server
+ curl -#fL -o ~/.cache/code-server/code-server_4.9.1_arm64.deb.incomplete -C - https://github.com/coder/code-server/releases/download/v4.9.1/code-server_4.9.1_arm64.deb
######################################################################## 100.0%
+ mv ~/.cache/code-server/code-server_4.9.1_arm64.deb.incomplete ~/.cache/code-server/code-server_4.9.1_arm64.deb
+ sudo dpkg -i ~/.cache/code-server/code-server_4.9.1_arm64.deb
Selecting previously unselected package code-server.
(Reading database ... 37391 files and directories currently installed.)
Preparing to unpack .../code-server_4.9.1_arm64.deb ...
Unpacking code-server (4.9.1) ...
Setting up code-server (4.9.1) ...

deb package has been installed.

To have systemd start code-server now and restart on boot:
  sudo systemctl enable --now code-server@$USER
Or, if you don't want/need a background service you can run:
  code-server

Deploy code-server for your team with Coder: https://github.com/coder/coder
```

Code server is now installed and we can get going!

## Code Server Setup

First thing we are going to do is to run code-server and get the default configuration file generated.

So in the home directory, we are going to run the code-server binary

```bash
code-server
```

If everything goes well, the output would look like this:

```bash
[2023-01-16T22:04:20.133Z] info  Wrote default config file to ~/.config/code-server/config.yaml
[2023-01-16T22:04:22.294Z] info  code-server 4.9.1 f7989a4dfcf21085e52157a01924d79d708bcc05
[2023-01-16T22:04:22.299Z] info  Using user-data-dir ~/.local/share/code-server
[2023-01-16T22:04:22.413Z] info  Using config file ~/.config/code-server/config.yaml
[2023-01-16T22:04:22.414Z] info  HTTP server listening on http://127.0.0.1:8080/ 
[2023-01-16T22:04:22.414Z] info    - Authentication is enabled
[2023-01-16T22:04:22.414Z] info      - Using password from ~/.config/code-server/config.yaml
[2023-01-16T22:04:22.415Z] info    - Not serving HTTPS
```

At this point we can't access code server yet because it's only listening on the IP address accessible by the raspberry pi, but we can change that easily now that the default config file has been generated at `~/.config/code-server/config.yaml`

Let's shut it down (`ctrl+c`), edit the config file and get going! Since we are in the terminal, probably the easiest text editor that we can use is `nano`:

```bash
nano ~/.config/code-server/config.yaml
```

First thing we want to do is:

* `bind-addr: 0.0.0.0:8080`: change the binding address so it can be accessible on our local network
* change the password to something easier to remember

Then hit `ctrl+x` to exit, `Y` to save the buffer and `return` key to save the changes in the current file

Let's relaunch code server and verify that we can now access it

```bash
code-server
```

You should see an output similar to this:

```bash
[2023-01-16T22:51:33.000Z] info  code-server 4.9.1 f7989a4dfcf21085e52157a01924d79d708bcc05
[2023-01-16T22:51:33.005Z] info  Using user-data-dir ~/.local/share/code-server
[2023-01-16T22:51:33.112Z] info  Using config file ~/.config/code-server/config.yaml
[2023-01-16T22:51:33.113Z] info  HTTP server listening on http://0.0.0.0:8080/ 
[2023-01-16T22:51:33.113Z] info    - Authentication is enabled
[2023-01-16T22:51:33.114Z] info      - Using password from ~/.config/code-server/config.yaml
[2023-01-16T22:51:33.114Z] info    - Not serving HTTPS 
```

Now open your browser and go to `http://codepi.local:8080`

You will be greated by a welcome message from code server and ask to type the password we just setup. Type it in and click the `SUBMIT` button

Give it a few seconds and you should see the startup screen

Everything looks good but obviously it will not start automatically... On raspberry pi we can do this by using a software called `systemd` (most linux distributions are using it)

First thing we need to do is to tell systemd how to start it with our config file

```bash
sudo nano /etc/systemd/system/code-server.service
```

We will add the following content:

```
[Unit]
Description=code-server
After=network.target

[Service]
User=codepi
Group=codepi

WorkingDirectory=/home/codepi
Environment="PATH=/usr/bin"
ExecStart=/usr/bin/code-server

[Install]
WantedBy=multi-user.target

```

Then hit `ctrl+x` to exit, `Y` to save the buffer and `return` key to save the changes in the current file

Let's enable our service to start on boot

```bash
sudo systemctl enable code-server
```

If everything goes well, you should see an output that looks like this

```bash
Created symlink /etc/systemd/system/multi-user.target.wants/code-server.service → /etc/systemd/system/code-server.service.
```

To start the server now, type the following

```bash
sudo systemctl start code-server
```

And to check that the service has started properly

```bash
sudo systemctl status code-server
```

## Getting rid of the "unsecure domains" warning

We will tell code-server to start in secure mode, it will give us a certificate. That certificate will not be recognized by our browser by default but there's a workaround.

First let's shut down our service

```bash
sudo systemctl stop code-server
```

Now we can change the config file

```bash
nano ~/.config/code-server/config.yaml
```

* update to `cert: true`
* add `cert-host: codepi.local`

Then hit `ctrl+x` to exit, `Y` to save the buffer and `return` key to save the changes in the current file

And let's restart the server now

```bash
sudo systemctl start code-server
```

Now go back to the browser, this time using [https://codepi.local:8080](https://codepi.local:8080)

You will see a warning telling you that your connection is not private. we will need to import the cert into chrome

1. Click the "not secure" button in the address bar
2. Click "certificate is not valid"
3. In the popup, go to the "details" tab and click "export" to save it on disk
4. double click on the downloaded cert and add it
5. navigate to chrome://settings/, Privacy and security, Security
5. double click on the new cert, under trust select trust all
7. open the following address in chrome [chrome://flags/#unsafely-treat-insecure-origin-as-secure](chrome://flags/#unsafely-treat-insecure-origin-as-secure)
8. add `https://codepi.local:8080` in the field and restart chrome

You should now have a secure version of cod-server accessible via https!