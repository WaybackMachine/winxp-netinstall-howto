**This is an automated howto guide for setting up a Linux box to install Windows XP from it via network, so that you'll never have to find that Windows CD again.**

There are many resources on the net for doing this, I just wrapped the whole procedure in a bunch of bash scripts so that you can do this in 10 minutes instead of 2 days like it took me.

## Status ##
  * v1.0 - April 27, 2011, tested with Debian 6.0.1, Windows XP SP3, ris-linux 0.4 DriverPack LAN RIS 10.11, DriverPack LAN 11.01

## Requirements ##

  * debian, but any linux should work with a bit of transposing
  * atftpd
  * udhcpd or other dhcp server
  * samba
  * a winxp iso (I used a winxp sp3 cd image)
  * [ris-linux](https://github.com/sherpya/ris-linux), for needed tools and patches and a BINL server (download and control scripts included)
  * LAN drivers from driverpacks.net, or windows drivers from the winxp iso, or both (download script included)
  * a7zip

## Preparation ##

  * download and unpack [winxp-netinstall-howto.tgz](http://winxp-netinstall-howto.googlecode.com/hg/winxp-netinstall.tgz), you'll find all the scripts inside
  * run `./get-ris-linux` to download the rislinux package
  * run `./patch-atftpd` to patch atftp with patch from rislinux
  * `adduser tftp`, which will also create `/home/tftp` where we'll put our boot files
  * edit `/etc/default/atftpd` adding `--user=tftp --group=tftp /home/tftp` to the `OPTIONS` variable
  * grab a winxp iso and fix the winxp.iso symlink to point to it (or replace the symlink with the actual iso file)
  * run `./mount-winxpcd` to mount that iso (or mount it yourself if its a real cd)
  * run `./extract-boot-files` to extract boot files from the iso
  * run `./patch-ntldr` to patch ntldr for netbooting
  * run `./get-lan-drivers` to download the lan drivers from driverpacks.net via bittorrent
  * run `./extract-lan-drivers` to extract downloaded driver files into the inf and drivers directories respectively
  * now is a good time to add more ethernet drivers if you need to, eg. I had to download the drivers for my Realtek GigE card from Realtek, others would not work; after you download your drivers, put any .inf files in the inf dir, and all other files (.sys, .dll, etc.) in the drivers dir
  * run `./build-drivers-database` to build nics.txt and drivers.cache by parsing the inf files from the inf dir
  * run `./build-mini-binlsrv` to build the mini-binl server that will tell windows about network drivers when it asks
  * run `./copy-winxp-files` to copy the i386 dir from the winxp cd along with the contents of the drivers dir
  * configure samba to share `/home/tftp/winxp` as the `reminst` share; take a look at `smb.conf.sample` for that
  * copy `winnt.sif.sample` to `/home/tftp/winnt.sif` and change the the network paths in there to point to your share
  * run `./fix-tftproot-perms` to make `/home/tftp` read-only for the tftp user under which atftp runs, very important!
  * configure your dhcp server for bootp: in addition to specifying an ip range to lease, you need to specify the the boot-file dhcp option, which in this case is `startrom.n12`

## Installation ##

  * restart samba, dhcpd and atftpd.
  * edit `home` in `mini-binlsrv` to point to our working directory
  * run `./mini-binlsrv start` to start the mini-binlr server.
  * configure client's bios for pxe booting and reboot it.
  * go to a Stop-and-Drop, insert a quarter, choose "slow and horrible", actually it doesn't matter what you choose, press "start".

To have this working between restarts of the server, just add or symlink `mini-binlsrv` to `/etc/init.d` and call `update-rc.d mini-binlsrv defaults` to have debian start the binl server automatically at boot time.

## Notes ##

The preparation process is mostly captured in scripts. Just run (and possibly edit
to configure) the scripts in whatever order. The scripts call dependent scripts automatically and they don't do the same thing twice. So in case of error, you have to remove the effects of the script in order to run it again, eg. you have to remove the drivers dir manually to be able to extract the drivers again.

When it comes to getting the drivers, you can also get them from the winxp iso itself, by running `./extract-winxp-drivers`.

If using pxelinux:
  * symlink `startrom.n12` to `startrom.0` (pxelinux makes assumptions based on file extension)
  * add a menu entry to `/home/tftp/pxelinux.cfg/default` with the line `kernel startrom.0`

## Resources ##

  * http://www.promodus.net/linuxris/
  * http://wiki.themel.com/XPNetInstall
  * http://echo.wordpress.com/2010/03/01/installing-windows-xp-from-network-ris/
  * http://diddy.boot-land.net/pxe/files/binl.htm
  * http://diddy.boot-land.net/pxe/files/ris_files.htm
  * http://diddy.boot-land.net/pxe/files/winnt_sif.htm
  * http://driverpacks.net/
  * http://oss.netfarm.it/guides/
    * http://oss.netfarm.it/guides/ris-linux.pdf
    * http://oss.netfarm.it/guides/ris-linux.php
    * http://oss.netfarm.it/guides/pxe.php
  * http://paradice.forumotion.com/t3-winntsif-lists-nearly-all-possible-settings-for-unattended

## Feedback ##
  * contact me at `cosmin.apreutesei@gmail.com`
