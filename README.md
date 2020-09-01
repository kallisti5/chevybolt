# Chevy Bolt EV Internals

This is a small collection of things i've discovered about the Chevy Bolt EV.

We haven't seen any OTA radio updates to the head unit, beatings will continue until
morale improves.

## Dealer TechLink

* [How to fix the Bolts tires](https://techlink.mynetworkcontent.com/wp-content/uploads/2018/03/GM_TechLink_05_March_2018.pdf)
* [How to check if hilltop reserve is set](https://techlink.mynetworkcontent.com/wp-content/uploads/2018/12/GM_TechLink_22_Mid-November_2018.pdf)
* [How to install the Bolts seat cushions](https://techlink.mynetworkcontent.com/wp-content/uploads/2019/05/GM_TechLink_08_Mid-April_20191.pdf)
* [2020 bolts have a new battery chemistry, and are replacable in five sections](https://techlink.mynetworkcontent.com/wp-content/uploads/2019/11/GM_TechLink_20_Mid-October_2019.pdf)
* [How to tell if the wiring harness is seated all the way](https://techlink.mynetworkcontent.com/wp-content/uploads/2020/02/GM_TechLink_03_February_2020.pdf)

## Radio

* Radio Receiver S-band HD 42583079
* Chevy Bolt EV OEM Radio Navigation 10" screen 42583079
  * Made by LG Innotek for GM
  * Model LC10S , LC10-S , LC10SB , LC10-SB
  * FCC "Silver box radio" https://fccid.io/BEJLC10SB
  * Supports Bluetooth, 2.4Ghz WIFI and 5 Ghz Wifi.
  * Wireless Android auto requires 5 Ghz wifi, so the head unit can "technically" be upgraded to it in the future.
* Seems to be QNX based
* Firmware updates are signed and look like the following:
  * indication.16008211.0 -- Checksum?
  * update/42490971.bin -- Unknown
  * update/42615022_CPU.bin -- Unknown.. Likely encrypted QNX filesystem
  * update/42615022.mnf -- Unknown.. Likely some manifest of what's in update package
  * update/42615022_OP.bin -- Unknown.. Likely more encrypted QNX updates
  * update/42615022.smd -- PEM Encryption keys, public. Checksums, sha256WithRSAEncryption

### Radio Reboot

If your head unit becomes locked up, press and hold "skip forward" and "home" for 5 seconds.

### Radio Firmware Updates

![GM Communication](images/gm_communication.webp)

* Radio performs a DNS lookup of vtmpub.oboservices.mobi
  * CNAME *USED* to be vtm-oboservices.nam1.lbx.gm.com (66.235.235.112, "Quality Technology Services")
  * CNAME *NOW* rcms-na-oboservices.nam1.lbx.gm.com (198.208.178.32, GM Subnet)
* TLS two way handshake, need a valid client TLS certificate to communicate
* Radio requests information from server (likely VIN tied)
* TLS guards from seeing any information about this request

### Man in the middle

![GM Communication](images/mitm.webp)

* Radio does not seem susceptible to man-in-the-middle attacks. TLS certificates are fully
  validated on the radio, and the request fails.

## Chevy Calibrations (aka, firmware updates)

Chevy releases new calibrations (software patches) for their cars, classically these have been to adjust things like engine timing. However, in an EV these updates adjust things like DCFC charging curves.
**20-NA-053** is a service bulletin for all 2017-2019 Chevy Bolt's released by GM to improve DCFC in cold weather.

To get this update, you have to head to the dealership, ask your service technician *NICELY* and trust that they'll actually apply it.
...or, you can apply it (and any future updates) yourself.

> I'm pretty sure GM offers acdelcotds.com to the public because of right-to-repair laws. They're just betting you don't know how to use it. Now you do!

### Terms

* SAE-J2534 - All new vehicles need to support these standards to reprogram systems which can include emmissions control changes.
* Tech2 - An older diagnostic system for GM vehicles. You don't need to mess with this.
* GDS2 - GM's current diagnostic system, which supports firmware updates via SAE-J2534
* Operating System - Base operating system of device.
* Calibrations - Applied on top of base operating system (like updates, patches)

### Tools Needed

* Windows 10 laptop of reasonable specifications
* Internet connection
* [GDS2 Adapter](http://www.vxdiagshop.com/wholesale/vxdiag-vcx-nano-for-gm-opel-wifi-version.html) -- $105
  * This is the one I use, it works fine and is a cheap Chinese clone.
* An account on https://www.acdelcotds.com/ and $40 per VIN gives you two years of firmware updates
  * You can search for updates for your vehicle before purchase by using [tis2web](https://tis2web.service.gm.com/tis2web)
  * This is the same tool Chevy dealers use. You can reprogram / re-vin an ECU, etc.

### Process

> WARNING: Dragons ahead. These steps can "brick" your entire car if performed improperly.

* Follow the instructions on your GDS2 adapter to set it up.
* Navigate to https://www.acdelcotds.com/ and follow the steps for "Vehicle Programming Software" / tis2web
* While choosing subsystems to program, here are some key points:
  * DON'T REPROGRAM SAFETY SYSTEMS! Anything body control, airbag, etc. Some of these require special steps.
  * Function: Generally "Programming"
  * Programming Type: "Normal"  ("VCI" is some special GM software drop for individual vehicles)
  * RPO Codes - Select the RPO code based on what is in your glovebox.
    * RPO IOB == Select this if your glovebox sticker lists "RPO"
    * RPO -IOB == Select this if your globebox sticker *doesn't* list "RPO"
* Follow the instructions *carefully* and perform all steps tds2web instructs you to.
* Don't interrupt programming!
