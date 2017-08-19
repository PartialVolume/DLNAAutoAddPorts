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
Write it to somewhere on your disc and give it a name ie DLNAAutoAddPorts.sh
Give it root ownership (chown root:root filename) and read+execute permissions
for user root only ie (chmod 500 filename).
As root, edit the root crontab (crontab -e) and add the following line changing 'linuxtv' to your own home directory.

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

