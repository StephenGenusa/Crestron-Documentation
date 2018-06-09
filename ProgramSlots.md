# Program Slots on Programmable Crestron Devices #

Crestron devices that take custom programming may have 1 or up to 10 "slots" available to put custom programming in. Some devices contain a single slot but may be upgraded to 10 slots with an additional license.

The concept of the slot is just a way for the Crestron tools to manage the different programs. Each has its own directory. 

Many of the current processors contain a slot 0 location which contains custom programming Crestron has created for the device. The application in slot 0 is booted first, to provide any services required to get the device operational, then slot 1 is booted, then slot 2 etc. 

If you are familiar with how operating systems work, there is nothing special about how custom programs run. They all share CPU time and it is possible for bad programming to hog CPU time such as creating a program where signals get propagated in an endless loop. Jamming signals on two programs that share an EISC, for example, would bring the processor to its knees.

