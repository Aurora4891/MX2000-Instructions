# MX2000-Instructions
My steps in debricking a bad Openwrt flash on a Linksys MX2000 using tftp on Debian.

Disclaimer: I am by no way an expert at debricking routers, using serial adapters or anything else in this writeup.
If you follow my steps and completely destroy your router I am NOT responsible for your actions.
My goal is to pay it forward and help someone in the future.

I have to thank georgem83 for helping me with this project and for all his work in making Openwrt work with the MX2000.

HARDWARE NEEDED: USB to TTL adapter. I used <a href="https://www.amazon.com/dp/B0D76NNWVW?ref=ppx_yo2ov_dt_b_fed_asin_title">these ones off Amazon</a>.

After a TON of googling, my first steps were to install the tftp server that will be used to serve the ITB file for booting the router from RAM and putty for communicating with the MX2000 over serial.

<code>sudo apt install tftpd-hpa putty</code>

Next you will need to download a couple files from the openwrt website. You will need the .itb file for booting from ram at:

<a href="https://downloads.openwrt.org/snapshots/targets/qualcommax/ipq50xx/openwrt-qualcommax-ipq50xx-linksys_mx2000-initramfs-uImage.itb">https://downloads.openwrt.org/snapshots/targets/qualcommax/ipq50xx/openwrt-qualcommax-ipq50xx-linksys_mx2000-initramfs-uImage.itb</a>

You will also need the .bin file that you want to use. I have put both the files I used on this repository. Luci is installed on the bin file here and they have been renamed for shorter typing.

The default path to files shared by tftpd-hpa is "/srv/tftp/". You will have to use sudo to copy the .itb file to "/srv/tftp"
If you don't want to use the default path for your firmware file you will need to change the settings of the tftp server to fit your needs using your editor of choice.

<code>sudo nano /etc/default/tftpd-hpa</code>

or

<code>sudo vi /etc/default/tftpd-hpa</code>

Change TFTP_DIRECTORY= to the directory you want to use and restart the tftpd-hpa service with:

<code>sudo systemctl restart tftpd-hpa.service</code>

scp -O /your/path/to/file.bin root@192.168.1.1:/tmp/firmware.bin
