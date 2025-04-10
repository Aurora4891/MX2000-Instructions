# MX2000-Instructions
My steps in debricking a bad Openwrt flash on a Linksys MX2000 using tftp on Debian (LMDE6).

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

Now you need to hook up the USB to TTL adapter so you need to open your router. Quoting georgem83...

<i>To open them, remove the rubber bands from the bottom to reveal the screws. Open them, then use a long thin non-sharp tool and push the top cover from the bottom from one of the corners with ample space. Remove the screws that connect the PCB/cooling elements from the casing and you can slide the cover off.</i>

Then hook the adapter to the serial pins on the router. GND to GND, RX to TX and TX to RX. Again quoting georgem83, the pinout on the router is...

<i>Pinout: GND -empty- RX - TX -empty (starting from GND label)</i>

In order to have to router get the .itb file you will need to use an ethernet cable and connect your computer to the 1st LAN port on the router. You also need to set the IP address of your computer to 192.168.1.254 and the subnet mask to 255.255.255.0 .

Next, plugin your USB to TTL adapter to your computer and open Putty. Select serial and in the Serial Line box type the path to your adapter. If your get the one I got, it will probably be "/dev/ttyUSB0". Next for speed type "115200". Now click the open button. I don't know if it matters, but while I was working with the router I turned off the Wifi on my laptop as to not cause unforseen problems.

Your now ready to turn on your router. You should now see a wall of text scroll past on the serial console and at some point, 
if your router fails to boot it will drop you back to a Uboot prompt. To load the .itb file and boot from RAM type:

<code>tftp $loadaddr name-of-your-file.itb && bootm $loadaddr</code>

After some time to boot up, you will be running Openwrt from RAM. DON'T turn off your router before you flash or you will have to run the above code again. You can now SSH into your router from a terminal window on your computer with:

<code>ssh root@192.168.1.1</code>

You will get a prompt about unknown connection, just type "yes" and hit ENTER.
Next you will have to send your firmware file to the router using SCP. Open another terminal window on your computer and run:

<code>scp -O /your/path/to/file.bin root@192.168.1.1:/tmp/firmware.bin</code>

Don't forget to change "/your/path/to/file.bin" to the actual path to your bin file. Now with your firmware file on the router you can flash your router. First you need to find your boot partition. To find the number of the current boot partition type:

<code>fw_printenv -n boot_part</code>

If you get "1" back, you need to install the firmware on the second partition using the command:

<code>mtd -r -e alt_kernel -n write /tmp/firmware.bin alt_kernel</code>

and in case of getting "2" back, run:

<code>mtd -r -e kernel -n write /tmp/firmware.bin kernel</code>

After it's done flashing, your router should reboot. If it still doesn't boot all the way you can power cycle the router a couple times until it boots to the good partition. You will have to leave it plugged in for a few seconds before you unplug it again to cycle the power. Once it boots you can follow the same steps above starting at SSH into root to flash the other partition.

As a quick note about my firmware file, I wanted Luci on my routers, but with the current images for our routers being snapshots they didn't come with Luci. Also a was setting them up without an internet connection so i couldn't use apk to install Luci. I used the Firmware Selector to make the bin file I used. The problem with it is that for some reason after a flashed it and went to run "fw_printenv -n boot_part" it came back with an error and not the boot partition number.

TLDR if "fw_printenv -n boot_part" throws an error, you can run this to fix it:

<code>echo "/dev/mtd9 0x0 0x40000 0x20000" > /etc/fw_env.config</code>

The last thing I have to say is, good luck!! Hopefully this will help someone down the road.
