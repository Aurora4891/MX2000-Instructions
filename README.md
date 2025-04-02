# MX2000-Instructions
My steps in debricking a bad Openwrt flash on a Linksys MX2000 using tftp on Debian.

Disclaimer: I am by no way an expert at debricking routers, using serial adapters or anything else in this writeup.
If you follow my steps and completely destroy your router I am NOT responsible for your actions.
My goal is to pay it forward and help someone in the future.

I have to thank georgem83 for helping me with this project and for all his work in making the MX2000 work with Openwrt.

After a TON of googling, my first steps were to install the tftp server that will be used to serve the ITB file for booting the router from RAM and putty for communicating with the MX2000 over serial.

<code>sudo apt install tftpd-hpa putty</code>

You will need to adjust the settings of the tftp server to fit our needs using your editor of choice.

<code>sudo nano /etc/default/tftpd-hpa</code>

or

<code>sudo vi /etc/default/tftpd-hpa</code>

