## DNLAAutoAddPorts

This script will open and close TCP & UDP ports automatically for specified processes. It was designed to open TCP/UDP ports as randomly used by DLNA/UPNP servers that need to be accessable on the local area network, while having most ports closed to to your LAN except those explicitly stated by UFW or IPTABLES etc. It was mainly written with bubbleupnpserver
and minidlnad in mind but should work for other DLNA software & devices. Any existing rules you have in your firewall are not affected.

It was designed to operate with IPTABLES & UFW, although it doesn't interact with UFW, UFW should be used
to enable the firewall, typically blocking incoming. DLNAAutoAddPorts.sh will then open the ports required
by the DLNA devices thus allowing port control based on the application and therefore not requiring a range
of ports to be opened manually (typically 32000-60000)or disabling the firewall to the LAN altogther. 

This script avoids the user having to manually open a range of ports from 32000-65000 for chromecast transcoding
 to function or to see bubbleupnpservers proxy servers.

Although the LAN is often considered 'trusted' this script is for those that like to have a
firewall running on their server blocking all incoming traffic from the LAN unless certain
ports are explicitly allowed. For instance if you are running UFW and have enabled it without adding any rules
all incoming ports will be blocked, all outgoing ports are open. You can then allow SSH, SMB etc. DLNA
appears to use random ports so you can't manually set an open incoming port. That's where DNLAAutoAddPorts.sh
comes into play.

So if you find that your DLNA/Chromecast devices don't work or can't be seen when your firewall is enabled
but work fine with your firewall disabled, this script may be for you.

## Installation:
Move DLNAAutoAddPorts.sh into a directory, in this example /home/linuxtv/bin/
Give it root ownership and read+execute permissions for root ONLY ..

`cd /home/linuxtv/bin`

`chown root:root DLNAAutoAddPorts.sh`

`chmod 500 DLNAAutoAddPorts.sh`

Open DLNAAutoAddPorts.sh with a text editor and find the line 

`processnames='minidlnad BubbleUPnPServer rygel'`

Remove or add process names as required, the default entries are minidlna, BubbleUPnPServer and rygel, if your
not running any of these servers feel free to remove them. Case is important ! If your not sure what process name
to enter or whether it's upper or lower case, makesure the process is running and use sudo netstat -anp to list
Active Internet connections (servers and established). At the top section of the listing you will see lines like this

`tcp        0      0 192.168.1.72:58051      0.0.0.0:*               LISTEN      1556/java       
tcp        0      0 0.0.0.0:8200            0.0.0.0:*               LISTEN      28287/minidlnad`

As shown above minidlnad is all lower case. BubbleuPnPServer is a special case because it's a java process. To find it's process name
type ..

`ps -ef | grep -i bubbleupnpserver`

and you will see a line like this ..

`root      1556     1  0 16:38 ?        00:01:26 java -Xss256k -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -jar BubbleUPnPServerLauncher.jar -dataDir /root/.bubbleupnpserver -httpPort 58050 -httpsPort 58051 -nologstdout`

The part of the line above that contains BubbleUPnPServerLauncher.jar is what we are looking for. We only actually use BubbleUPnPServer, but if you use another java based DLNA server
then you would look for the .jar bit. ie AnotherDLNAServer.jar. You could then add that to the process name list.

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
unless it's encountered an error, to find out what it's doing add -v. It's advisable NOT to add
-v to the crontab entry, because if you are running an MTA such as sendmail the cron will send a mail to
/var/spool/mail/root every minute. This might be useful for debugging but it will occupy disc space until
you delete /var/spool/mail/root

