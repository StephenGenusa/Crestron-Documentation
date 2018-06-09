# Best Practices with Crestron #

----------

### Device Firmware and Upgrading ###
 - There is probably a 50/50 split about upgrading firmware and as such, the best practice depends on which Crestron employee you ask. Some Crestron employees will tell you very plainly, if you aren't experiencing problems, or you don't require new features, firmware should not be upgraded for the sake of upgrading. They say, "if it ain't broke, don't fix it" which is a euphemistic way of saying, if it isn't broken, don't break it. 
 - Others argue that upgraded firmware, as a rule, contains bug fixes and you should always upgrade.
 - At the same time, one must recognize that upgraded firmware may introduce new bugs and problems that you haven't seen previously. 
 - You will find that Enterprise users with a sound best practices policy have a **do not upgrade firmware policy on fully developed and tested systems**. Once a system is thoroughly tested, the firmware version is frozen. If it works, there's no reason to break it. Freezing the compiler and database versions is also necessary if you are going to retain control and stability. Crestron has made breaking changes to the database and the compiler with little or no notice.
 - **Always read the release notes associated with the update.** They do not contain a complete change history but they are always worth reading. 
 - **Upgrading on release day is not a best practice unless you are working directly with Crestron on resolving a particular bug.**
 - My personal recommendations are:
  - Upgrade all devices on a new job to the current firmware revision. Build and thoroughly test the system.
  - If the device is a proc and the system has been stable for 6 months, the firmware should not be changed. If the proc exhibits the same instability every 3-6 months, I upgrade the firmware. If program changes are made, and you are on the latest databases and compiler, you may need to update the firmware.
 - **Avoid using the PUF tool unless the device you are upgrading requires it.** The PUF utility is known to brick devices. If you use the PUF utility, only use it on devices where file transfer/the PUF command are not supported. If you must use the PUF tool verify that you have a solid connection and a stable network before proceeding. Pinging the device for a while to make sure there are no lost packets would be one way of testing the network stability: "ping [ip] -t" and then CTRL-c to stop the ping when you are satisfied.
 - **Best practice is to FTP the file onto the device and issue the PUF command, on devices where the PUF command is supported.**
 - Jumping from 5 year old firmware to the very latest may lead to problems. I recently had a case where a Series-3 proc was on 2013 firmware and I was updating to 2018 firmware. The institutional client had requested all devices be updated to the latest firmware. The proc did a partial update and then went into an endless reboot cycle, doing an analysis (read) of the updates, writing out final updates, and rebooting again. I pulled power during the analysis phase so as to minimize the chance of corrupting the file system. The system returned to a normal boot cycle. I deleted the 2018 firmware, did a file transfer of 2015 firmware, issued the puf command. Updates completed normally. Deleted the 2015 puf, uploaded the 2018 puf, issued the puf command and updates completed successfully.

----------
### Cresnet Best Practices ###
 - Crestron recommends that you keep the total number of devices on one segment or leg of a Cresnet run to a maximum of 20 devices. Some people put over 100 devices on a Cresnet segment. There are two issues with this: the RS-485 stack communicates in a round-robin fashion. By the time it cycles back around to the devices that have traffic, latency can become an issue.  Larger Cresnet networks can also introduce intermittent problems that are hard to troubleshoot. The new Crestron bridge devices explicitly limit the number of devices you can put on a run to less than 30.

----------
### Networking Best Practices ###
 - Isolate the controls network from other networks to the extent possible.
 - Develop a network standards document to *segregate* device types on the subnet(s) involved. For example: first you would have routing devices, some empty ip slots, then network switches, again some empty IP slots, processors, empty IP slots, touchpanels, etc... with a DHCP range at the bottom of the subnet. This makes device location, troubleshooting and maintenance much easier (and is consistent with Cisco best practices).

----------
### Security Best Practices and Notes ###
 - **Best practice is to run the Toolbox security audit tool** to determine what steps can be taken to tighten up security on Crestron processors. Crestron has added a device security audit tool which is a huge step in the right direction, so far as detailing what steps can be taken to help secure some devices. 
 - **The RS-485 protocol used over Cresnet is not secure** contrary to certain claims. Because of the way these networks are wired, tapping into a Cresnet network and either sniffing controls signals to profile a system or sending out spoofed control signals is trivial, if you have physical access to the wiring or the equipment.
 - Using either CIP via Cresnet or CIP via Ethernet it is easy to locate and spoof IP ID's and manipulate signals values without having any insider knowledge of the system. This means, the barrier into manipulating/controlling a control system is very low.
 - Even devices with SSH enabled are inherently insecure when a default user name of Crestron with no password is the default.

----------
### Pyng-Hub Best Practices ###
 - **Get the Pyng-Hub on the Internet and the Crestron Portal *before* you start configuring the client system.** This is very important because you cannot copy the DAT files directly from the Pyng-Hub file system as a restorable backup. Copying the files back to Pyng-Hub with File Manager is a roll of the dice situation that I have seen fail. The only effective way to backup the system config and prevent data loss is through the portal. I have seen **partial loss of recent system configuration changes due to issuing the reboot command from Toolbox Console**. Make it a habit to manually push the current configuration to the cloud *before* disconnecting power or rebooting.
 - **Put all Pyng-Hub devices on a UPS**. I have seen various scenarios where a loss of power to the device resulted in full or partial loss of the system configuration. A full backup through Crestron's portal is the only effective way to recover and not lose time in rebuilding the system configuration.
 - **After making any system configuration changes to a Pyng-Hub, manually push a backup from the GUI.**


----------
### SIMPL Windows Programming Best Practices ###

#### 1. Program Name ####
#### 2. Program Organization ####
#### 3. Signal Naming ####
Signal naming is one of the most overlooked but most important elements in developing maintainable code. As in any programming, it's important to be able to go back 6 months later and in a short period of time be able to determine what the intent of the program logic is. For example, the signal name should indicate source and destination of signals and other characteristics such as whether it is coming off base symbols such as buffer, oscillator or a one shot. In a multi-room system, having the room number or other identifying information makes reading, debugging and troubleshooting *much* easier. Don't make someone else's brain explode because you just want to slap signals here and yonder and the signal names indicate nothing of practical value. It is the sign of an experienced Crestron programmer when he's slapping signals around like he's playing whack-a-mole but not necessarily a sign of competence.

#### 4. Comments ####
#### 5. Documenting Changes ####

----------
### VTPro-e Best Practices ###
#### 1. Project Name ####
#### 2. Project Organization ####
#### 3. Indicating State ####

Indicating state through color alone is not a best practice. Color blind users will not be able to operate the system.
