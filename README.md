# pogoplug_mobile_uboot_installer
Boot a **Pogoplug Mobile** or **Pogoplug Series 4** into Linux from USB or SD media by installing [uBoot](https://en.wikipedia.org/wiki/Das_U-Boot).

## TL;DR:

```
cd /tmp
wget http://ssl.pepas.com/pogo/uboot.sh
ash uboot.sh
```

## Unlock the cheapest Linux server on the planet!

Meet the **Pogoplug Mobile**, a **[$10 Linux server](http://www.amazon.com/Pogoplug-Backup-Sharing-Discontinued-Manufacturer/dp/B005GM1Q1O)**:

![](https://raw.githubusercontent.com/pepaslabs/pogoplug_mobile_uboot_installer/master/.github_media/Pogoplug.jpg)

### What you get for ten bucks:

![](https://raw.githubusercontent.com/pepaslabs/pogoplug_mobile_uboot_installer/master/.github_media/Pogoplug_Mobile_Rear.jpg)

* **800**MHz **ARM**v5TE processor
* **128**MB of **RAM**
* an **Ethernet** port
* a **USB** 2.0 slot
* an **SD** card slot
* 128MB of built-in flash

Also included with the Pogoplug:
* 12V/1A power adapter
* 3 foot ethernet cable

**UPDATE:** I have verified this script also works with the **[Pogoplug Series 4](http://www.amazon.com/Pogoplug-Series-4-Backup-Device/dp/B006I5MKZY)**.  Twice the price, with the addition of a SATA port and two addition USB ports (and a 12V/2A power adapter).

## Running the script

So, you ordered a Pogoplug Mobile from [amazon.com](http://www.amazon.com/Pogoplug-Backup-Sharing-Discontinued-Manufacturer/dp/B005GM1Q1O) and it just arrived.

Here's what you do:

1. **Plug it into your local network** and turn it on (connect the power supply).

   The Pogoplug will boot its busybox-based Linux install and try to grab a DHCP address.

2. **Find its IP address.**

   Your router assigns IP addresses through DHCP. Check your logs, or alternatively, use [`nmap`](https://nmap.org/). For example, if your subnet prefix is `192.168.1`,
   
   `nmap -sn 192.168.1.* | grep PogoplugMobile`
   
   The latter part sorts out all responses that do not include `PogoplugMobile`:
   
   ```
Nmap scan report for PogoplugMobile.localnet (192.168.1.204)
   ```

3. **Start SSH** on the Pogoplug

   The Pogoplug's stock busybox-based Linux distro ships with the dropbear SSH daemon installed, but it isn't running by default.  Luckily, you can start it by sending an HTTP POST to the Pogoplug's web interface.
   
   If your local network supports resolving the Pogoplug's default hostname of `PogoplugMobile`, you can run this curl command:
   
   `curl -k "https://root:ceadmin@PogoplugMobile/sqdiag/HBPlug?action=command&command=dropbear%20start"`
   
   Otherwise, you'll have to stick its IP address in there, e.g.:
   
   `curl -k "https://root:ceadmin@192.168.1.204/sqdiag/HBPlug?action=command&command=dropbear%20start"`
   
   The curl command will spit out a bunch of HTML in your terminal.  That means it worked.
   
4. **SSH into the Pogoplug**

   Again, if you can resolve the Pogoplug's default hostname, run this:
   
   `ssh root@PogoplugMobile`
   
   Otherwise, use its IP address, e.g.:

   `ssh root@192.168.1.204`
   
   The stock root password is `ceadmin`.
   
5. **Download and run the script**

   ```
cd /tmp
wget http://ssl.pepas.com/pogo/uboot.sh
ash uboot.sh
```

   Here's what the script does (have a look: http://git.io/vCtIl):
   * Prompt you to verify the Pogoplug's MAC address
     * (it is printed on a sticker on the underside of your Pogoplug)
   * Download some flash utility binaries
   * Download some uboot binary blobs
   * Burn uboot into flash
     * (it will prompt you for permission to do this)
   * Burn uboot's default settings into flash
     * (it will prompt you for permission to do this)
   * Tweak a bunch of firmware settings in the flash
   * `poweroff` the Pogoplug
     * (it will prompt you for permission to do this)

**After rebooting, your Pogoplug will be able to do the following:**
* Boot the stock Pogoplug OS
  * (just disconnect any USB drive and SD card and turn it on)
* Boot your Linux distro of choice from USB or SD card
  * (uboot expects a bootable ext3 partition labelled 'rootfs')

**Note:** If you are rebooting back into the stock Pogoplug Linux install, don't freak out when you can't ssh into it anymore.  You have to run that `curl` command again every time it boots into its stock OS.

**Note:** If you are running this script on several Pogoplugs and get tired of answering all of the prompts, you can instead tell the script to assume yes to all prompts via `ash uboot.sh -y`.

## FAQ

**Q: Why aren't you using a github URL in the above instructions?**

A: It turns out the version of wget (busybox) which ships with the Pogoplug doesn't do HTTPS at all, and github now forces all HTTP traffic onto HTTPS, which causes wget to fail like so:

```
# wget -O - http://git.io/vCtIl
Connecting to git.io (54.225.117.235:80)
wget: not an http or ftp url: https://raw.githubusercontent.com/pepaslabs/pogoplug_mobile_uboot_installer/master/pogoplug_mobile_uboot_installer.sh
```

**Q: I'd like to verify the contents of the script before running it.**

A: Run this:

```
md5sum -c - << EOF
c583afbc0e503b77db2eba3ad927b345  uboot.sh
EOF
```

## Credits and changelog

The uboot work was done by Bodhi and others at the [Doozan uboot forums](http://forum.doozan.com/list.php?3), and the script is based on [Qui Hong's 2014 Pogoplug tutorial](http://blog.qnology.com/2014/07/hacking-pogoplug-v4-series-4-and-mobile.html).

### 2016-09

* Updating (in progress) to use [Bodhi's 2016.05 release](http://forum.doozan.com/read.php?3,12381), [Debian w/ kernel 4.4.0](http://forum.doozan.com/read.php?2,12096)

### 2015-10

* Initial version of script,  verified to work with the **Pogoplug Mobile** (models **POGO-V4-A1-01** and **POGO-V4-A1-05**) and **Pogoplug Series 4** (model **POGO-V4-A3-01**).


## Related github projects

* [pogoplug-v4-bodhi-rootfs-debian](https://github.com/pepaslabs/pogoplug-v4-bodhi-rootfs-debian)
* [pogoplug_static_binaries](https://github.com/pepaslabs/pogoplug_static_binaries)
