<body>

<BODY BGCOLOR="#333333">
   
  <h1>Recover-Mac-In-VM</h1>
  <p>How to recover or revive a Mac with a macOS VM in Linux QEMU</p>

  <h2>You will need:</h2>
  <ol>
    <li>A Mac that needs to be recovered or revived (I have only tested with Apple silicon)</li>
    <li>A USB-C to USB-C cable. The Macs are very particular about what cables you use so you <em>can</em> try using other cables (ie, USB-C to USB-A) but know this can cause problems</li>
    <li>A second laptop or PC running Linux (preferably with a USB-C port. I haven’t tested USB-C to USB-A cables or hubs.)</li>
    <li>A macOS vm of Monterey 12.4 or newer in QEMU. There are projects on GitHub that automate the process of installing macOS in a Linux VM, but just know that this guide will focus on a QEMU KVM Libvirt VM.</li>
    <li>The ability to do trial and error</li>
  </ol>

  <h2>Getting Started</h2>
  <p>In your macOS VM you will need to install Apple Configurator 2. With that installed, we can move right along.</p>

  <p>In order to use your macOS VM to recover or revive your Mac, you will need to connect your mac to the second PC running the macOS VM. Follow the instructions on <a href="https://support.apple.com/guide/apple-configurator-mac/revive-or-restore-a-mac-with-apple-silicon-apdd5f3c75ad/mac">Apple's website</a> as to the proper way to connect your Mac (the ports are specific). Once your Mac is connected to your PC, you will need to pass through the Mac to the VM so macOS can see the device and do its magic. The easiest way to do this is to pass through the USB device that the Mac identifies as to the VM.</p>

  <h2>USB Pass Through Madness</h2>
  <p>Passing though a USB device to a VM is very simple if you are using QEMU.</p>
  <p>The first thing you need to do is identify the USB device and get its vendorID and productID. To do this, open up a terminal and type "lsusb" (without quotations). This will list all detected USB devices in the following format:</p>
  <pre>
Bus XXX Device XXX: ID XXXX:XXXX Device description
  </pre>
  <p>With the Bus, Device, and ID being any number of different numbers and/or letters. This is what my USB drive shows up as:</p>
  <pre>
Bus 004 Device 001: ID 1d6b:1205 ASmedia mass storage
  </pre>
  <p>When passing through a USB device to a VM in QEMU, the most reliable way is to assign it based on the VendorID and productID, as those don’t normally change. In my case, the vendorID is "1d6b" and the productID is "1205". Yours will be different.</p>
  <p>With these two IDs, we can now pass through the USB drive to the VM with this QEMU launch option:</p>
  <pre>
-device usb-host,vendorID=0xXXXX,productID=0xXXXX
  </pre>
  <p>Notice how you need to add 0x in front of both IDs. There is only one space, and it is between "-device" and "usb-host,vendorID=0xXXXX,productID=0xXXXX." In my case passing though my USB drive would look like:</p>
  <pre>
-device usb-host,vendorID=0x1d6b,productID=0x1205
  </pre>
  <p>Booting up the VM, the USB device should show up and work normally in it. Verify by unplugging it, making sure the VM detects it's unplugged, and plugging it into different ports. The device should show back up in the VM. It is important that this "hot plugging" works, as the Mac you want to restore will be restarted multiple times by the VM and be put into different USB modes.</p>

  <h2>Mac different USB modes</h2>
  <p>During this process, I have observed at least 3 different USB modes that a Mac will go through during the recovery process. They are "DFU" "Recovery" and "Normal," but there may be more on your Mac. You can see these different modes when you run "lsusb" with the Mac connected to the PC. It will show up as something along the lines of:</p>
  <pre>
Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device (UDF mode)
Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device (Recovery)
Bus XXX Device XXX: ID XXXX:XXXX Apple Mobile Device
  </pre>
  <p>My testing was on an m1 MacBook Pro, so your device may show as something slightly different. However, as you can see, lsusb will tell us what mode the device is in. While the vendorID will stay the same, the productID will change, so you will need to pass through all 3 of these modes independently. Simple enough, you will just need 3 -device usb-host,vendorID=0xXXXX,productID=0xXXXX arguments in QEMU.</p>
  <p>To get the vendorID and productID for DFU and Recovery mode, you just have to put the Mac in its respective mode with it connected to the PC and run lsusb. Getting it in "normal" mode will be a bit more difficult if the machine is completely unable to boot. In that case, you can have the first two modes passed through to the VM and start the recovery process. Once you are on step 4 of the Apple configurator recovery process, start running lsusb in another window on the host until it changes from "Apple Mobile Device (Recovery)" to just "Apple Mobile Device." Remember the productID, and make your third pass-through argument.</p>
  <p><strong>NOTE:</strong> I've only tested on the 2020 m1 MacBook Pro, so there may be variances in how many different modes the Mac will cycle through. I don’t see why there needs to be more than three though.</p>

  <h2>Recovering the Mac</h2>
  <p>Once all of your different modes have been passed through to the VM, just follow <a href="https://support.apple.com/guide/apple-configurator-mac/revive-or-restore-a-mac-with-apple-silicon-apdd5f3c75ad/mac">Apple's instructions</a> on how to recover or revive your Mac using their Apple configurator. Then ta-da! You now have a working mac! </p>

</body>
