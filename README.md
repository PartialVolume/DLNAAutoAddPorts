## Purpose

This script will automatically open and close TCP/UDP ports for user specified processes within 1 second of the process using a particular port.

## Scope

Any Linux based server running BubbleUPnPServer/minidlna/rygel with IPTABLES and UFW firewall. May also be suitable for use with other DLNA/UPnP servers such as plex (untested). It requires iptables, netstat, cut which are installed as standard on ubuntu.

## The Problem This Script Resolves

Firstly, a lot of people like to run a firewall on their Linux server, with all incoming traffic denied. It then becomes a routine habit to expect to manually open a port for a particular service, ie 22 for SSH, 25 for SMTP etc.. so when they install their new DLNA/UPnP media server/transcoder, ie bubbleupnpserver they read the docs & search the internet for what ports they are supposed to open in their firewall. In the case of bubbleupnp that's 1900/udp, 8200/tcp, 58050/tcp and 58051/tcp (unless you have changed those later two ports in /etc/default/bubbleupnpserver). You will start streaming your content but then find that the transcoding that the server is supposed to do ..doesn't, maybe some files won't play on your TV (the transcoder should automatically take care of that ... and so after much searching you realise that it all works properly if you disable your firewall. Or maybe even opened up 20000 ports from 30000-50000. Of course anybody that is a bit particular about which ports are opened, looks upon opening up that many ports just so their DLNA will work properly with horror ! If you're lucky you may have come across a snippet of information that explains the random nature of port usage that DLNA/UPnP untilizes.. and that's the problem. Each time bubbleupnpserver starts it will use a random port somewhere between ~30000-60000. Reluctantly you realise you need to open a range of ports for it to work. Maybe you've looked at UFW/Iptable or Redhats firewall but couldn't find a means by which they would open a random port for a process, service maybe, but then again fixed ports.. what you need is a process specific firewall that's able to open a port for bubbleupnpserver, unfortunately none seem to do exactly what you want... At least that's what I've found, correct me if you know another way !

So after a couple of years, I'd had enough, I wanted my incoming ports closed except for those services (SSH) that I wanted open and it was OK if bubbleupnpserver opened random ports but I wanted control over it and would only open the specific port it required not a whole range of ports.

So I wrote this script that monitors what ports bubbleupnpserver, minidlna, rygel (& other DLNA servers) use in real time and open and close those ports only for those particular processes. So now I've got control of my firewall while having a fully functional bubbleupnpserver with transcoding that works and proxy servers that I can make use of as required.
 
So if you find that your DLNA/Chromecast devices don't work or can't be seen when your firewall is enabled, proxy servers that don't appear in your library and all this works fine with your firewall disabled, this script may well be for you.

## Installation:
Move DLNAAutoAddPorts.sh into a directory, in this example /home/linuxtv/bin/
Give it root ownership and read+execute permissions for root ONLY ..

`cd /home/linuxtv/bin`

`chown root:root DLNAAutoAddPorts.sh`

`chmod 500 DLNAAutoAddPorts.sh`

Open DLNAAutoAddPorts.sh, search for a user configuration section which includes the following lines that can be edited as required.

```processnames='minidlnad BubbleUPnPServer rygel'
min_DLNA_port='32000'
allowed_TCP_ports_below_min_DLNA_port='8200'
allowed_UDP_ports_below_min_DLNA_port='1900 5353'
```
**processnames='minidlnad BubbleUPnPServer rygel'**
Remove or add process names as required, the default entries are minidlna, BubbleUPnPServer and rygel, if your
not running any of these servers feel free to remove them. Case is important ! If your not sure what process name
to enter or whether it's upper or lower case, makesure the process is running and use sudo netstat -anp to list
Active Internet connections (servers and established). At the top section of the listing you will see lines like this

`tcp        0      0 192.168.1.72:58051      0.0.0.0:*               LISTEN      1556/java`
 
`tcp        0      0 0.0.0.0:8200            0.0.0.0:*               LISTEN      28287/minidlnad`

As shown above minidlnad is all lower case. BubbleuPnPServer is a special case because it's a java process. To find it's process name
type ..

`ps -ef | grep -i bubbleupnpserver`

and you will see a line like this ..

`root      1556     1  0 16:38 ?        00:01:26 java -Xss256k -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -jar BubbleUPnPServerLauncher.jar -dataDir /root/.bubbleupnpserver -httpPort 58050 -httpsPort 58051 -nologstdout`

The part of the line above that contains BubbleUPnPServerLauncher.jar is what we are looking for. We only actually use BubbleUPnPServer, but if you use another java based DLNA server
then you would look for the .jar bit. ie AnotherDLNAServer.jar. You could then add that to the process name list.

**min_DLNA_port='32000'**
Minimum lower port for use by DLNA for random ports, I'm not sure what this should be, but seems to be about 32000, all random ports I've seen DLNA programs use are somewhere between 32000 & 60000

**allowed_TCP_ports_below_min_DLNA_port='8200'**
All TCP ports below 'min_DLNA_port' will not be opened except for these exemptions

**allowed_UDP_ports_below_min_DLNA_port='1900 5353'**
All UDP ports below 32000 will not be opened except for these exemptions


## Edit root crontab to run script every 60 seconds
As root, edit the root crontab ..

`crontab -e`

 and add the following line changing 'linuxtv' to your own home directory.

**`* * * * * /home/linuxtv/bin/DLNAAutoAddPorts.sh`**

This crontab entry will execute DLNAAutoAddPorts.sh every minute, so worst case if the bubbleupnpserver or DLNA server
software is restarted or new servers or proxy servers added it may take 60 seconds before the firewall is opened.

## Troubleshooting:
If you want to see which ports are currently open, type more /tmp/ports*, this can also be confirmed
by typing iptables -S INPUT. If your running UFW, the ufw status command will not show these ports as
open, however they are open.

DLNAAutoAddPorts.sh when run without any arguments does not ouput any information to stdout,
unless it's encountered an error, to find out what it's doing add -v. Even though DLNAAutoAddPorts.sh
may be run every minute by the cron, it's ok to run the command manually to see what it's doing.

`cd /home/yourname/bin`

`sudo DLNAAutoAddPorts.sh -v`

```DLNAAutoAddPorts V2.0.5
Ports are being checked/opened/closed for the following processes:
minidlnad
BubbleUPnPServer
rygel
Ports identified as being used by the above processes..
TCP/40985 58050 58051 8200
UDP/1900 40527 49929 5353
diff output of previous and current port logs follows..
No change in TCP ports
diff output of previous and current port logs follows..
No change in UDP ports
```

 It's advisable NOT to add
-v to the crontab entry, because if you are running an MTA such as sendmail the cron will send a mail to
/var/spool/mail/root every minute. This might be useful for debugging but it will occupy disc space until
you delete /var/spool/mail/root

