# Recover-Mac-In-VM
How to recover or revive a Mac with a macOS VM in Linux QEMU

You will need:

  1. A Mac that needs to be recovered or revived (I have only tested with Apple silicon)

  2. A USB-C to USB-C cable. The Macs are very particular about what cables you use so you *can* try using other cables (ie, USB-C to USB-A) but know this can cause problems

  3. A second laptop or PC running Linux (preferably with a USB-C port. I haven’t tested USB-C to USB-A cables or hubs.)

  4. A macOS vm of Monterey 12.4 or newer in QEMU. There are projects on GitHub that automate the process of installing macOS in a Linux VM, but just know that this guide will focus on a QEMU KVM Libvirt VM.

  5. The ability to do trial and error

  #Getting Started

  In your macOS VM you will need to install Apple Configurator 2. With that installed, we can move right along. 
  
  In order to use your macOS VM to recover or revive your Mac, you will need to connect your mac to the second PC running the macOS VM. Follow the instructions on [Apple's website](https://support.apple.com/guide/apple-configurator-mac/revive-or-restore-a-mac-with-apple-silicon-apdd5f3c75ad/mac) as to the proper way to connect your Mac (the ports are specific). Once your Mac is connected to your PC, you will need to pass though the Mac to the VM so macOS can see the device and do its magic. The easiest way to do this is to pass through the USB device that the Mac identifies as to the VM.

#USB Pass Through Madness

  Passing though a USB device to a VM is very simple if you are using QEMU. 

  The first thing you need to do is identify the USB device and get it's vendorID and productID. To do this, open up a terminal and type "lsusb" (without quotations). This will list all detected USB devices in the following format:

Bus XXX Device XXX: ID XXXX:XXXX Device description

  With the Bus, Device, and ID being any number of different numbers and/or letters. This is what my USB drive shows up as:

Bus 004 Device 001: ID 1d6b:1205 ASmedia mass storage

  When passing through a USB device to a VM in QEMU, the most reliable way is to assign it based on the VendorID and productID, as those don’t normally change. In my case, the ventorID is "1d6b" and the productID is "1205". Yours will be different.

  With these two IDs, we can now pass though the USB drive to the VM with this QEMU launch option:

-device usb-host,vendorID=0xXXXX,productID=0xXXXX

  Notice how you need to add 0x in front of both IDs. There is only one space, and it is between "-device" and "usb-host,vendorID=0xXXXX,productID=0xXXXX." In my case passing though my USB drive would look like:

-device usb-host,vendorID=0x1d6b,productID=0x1205

  Booting up the VM, the USB device should show up and work normally in it. Verify by unplugging it, making sure the VM detects its unplugged, and plunging into a different ports. The device should show back up in the VM. It is important that this "hot plugging" works, as the Mac you want to restore will be restart multiple times by the VM and be put into different USB modes.

And on that subject...

#Mac different USB modes

  During this process, I have observed at least 3 different USB modes that a Mac will go through during the recovery process. They are "DFU" "Recovery" and "Normal" but there may be more on your Mac. You can see these different modes when you run "lsusb" with the Mac connected to the PC. It will show up as something along the lines of:

Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device (UDF mode)

Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device (Recovery)

Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device 

  My testing was on a m1 MacBook Pro so your device may show as something slightly different. However, as you can see, lsusb will tell us what mode the device is in. While the vendorID will stay the same, the productID will change, so you will need to pass through all 3 of these modes independently. Simple enough, you will just need 3 -device usb-host,vendorID=0xXXXX,productID=0xXXXX arguments in QEMU. 

  To get the vendorID and productID for DFU and Recovery mode, you just have to put the Mac in its respective mode with it connected to the PC and run lsusb. Getting it in "normal" mode will be a bit more difficult if the machine is completely unable to boot. In that case, you can have the first two modes passed through to the VM and start the recovery process. Once you are on step 4 of the apple configurator recovery process, start running lsusb in another window on the host until it changes from "Apple Mobile Device (Recovery)" to just "Apple Mobile Device." Remember the productID, and make your third pass through argument.

  NOTE: Ive only tested on the 2020 m1 MacBook Pro, so there may be variances in how many different modes the Mac will cycle through. I don’t see why there needs to be more than three though.


#Recovering the Mac

  Once all of your different modes have been passed through to the VM, just follow [Apple's instructions](https://support.apple.com/guide/apple-configurator-mac/revive-or-restore-a-mac-with-apple-silicon-apdd5f3c75ad/mac) on how to recover or revive your Mac using their Apple configurator. Then ta-da! you now have a functioning Mac again!
