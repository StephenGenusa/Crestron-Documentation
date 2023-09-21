# Avoiding RMAs by Making Local Repairs #

If your device is in-warranty, it's going to only make sense to return it for repair but over the years equipment ages and there are some repairs you can make locally, avoiding the costs of RMA'ing the equipment, especially if you prepare.

# WARNING #
Crestron equipment, particulary new equipment is made with a lot of static sensitive SMD devices. **The reader and user of the following information is liable for any damages incurred by the use or misuse of the following information.**

If you destroy your equipment by opening it, that's your responsibity and liability, not mine. The following information is for those with a functioning knowledge of electronic equipment and how to properly handle it.
# WARNING #


## General Comments about Boot SDCARDs: ##

- Most of the complex equipment that boots some operating system like touch panels and procs have an SDCARD inside them. These are not the SDCARDs you can externally plugin on some devices for extra space. These are located inside the device so at least one set of screws will likely have to come off to open the box. Sometimes the card is hidden underneath a metal shield. You'll need to remove a screw or two to get to these
- These cards are partioned like any boot device and the partitions depend on the O/S the device is running
- SDCARDs go bad after 3-5 years, especially if you have the device on any kind of boot schedule. The quality of the cards I've seen have varied but are generally offbrand cheap SDCARDs you probably wouldn't buy yourself
- An easy though incomplete way to test if the SDCARD is bad is to DD image the card. If you get 800 MB off a 4 GB card you know the card is bad
- Sometimes a device will boot and you can push a fresh firmware file, run PUF successfully and then days or weeks later the device is down again. This may be a sign that the SDCARD is on the point of failure
- If you image a working sdcard to a file, or image it from a similar model, you can repair many devices by replacing the SDCARD. You want to use DD on Linux or OS X to create an image file from a functioning device's SDCARD and then image it to a new SDCARD.
- Where you have families of devices, say the DSP-1281, you can take a DSP-1283 imaged card and put it into a DSP-1281, and the device will determine at bootup what kind of DSP device it is and boot as a DSP-1281, not a DSP-1283. My assumption, which has been limited but correct so far, is that if the device is in the same firmware release, you can safely swap cards
- **My practice, and I highly recommend this: get the device up and running and then do either an upgrade or a forced upgrade using PUF with the latest firmware. Because there are device variations the base image file you have may be lacking software that would otherwise be installed during the PUF process**
- The older the device the smaller the SDCARD. Not a problem. The partition setup will be identical since you are using DD. For example, say you have a device you need to get up the same day and the local big box store only has 32 GB cards in stock, but you have a bad 4 GB card. No problem. Just take a working 4 GB image and DD it to the 32 GB card. Yes, firmware updates work. The O/S will only see the normal 4 GB partitions and doesn't know the difference
- Once the card is replaced, and the firmware updated, you can do a INIT, and then reload the DSP config or the proc's programs, and you are back in business
- Having a library of DD file images for various devices is going to serve you well

## Other Repairs: ##
- Various equipment, especially procs are going to have onboard batteries to keep the system clock running. The Pac2M has a CR2320 battery underneath the proc board. You may want to replace this on some schedule for clients. If the device clock drops the correct date/time during a power outage you are going to know the battery is no longer providing sufficient voltage to the board to run the system clock
- Power supplies - If the unit has a power supply unit it may be an OEM unit you can purchase from an major electronics dealer. I had a series 3 proc that would not power up. I used a voltmeter to check the power supply and found it was dead. I looked for a product ID number, found it came from a major electronics parts retailer, ordered a new one and installed it for substantially less than an RMA would have cost. It was the exact same power supply
