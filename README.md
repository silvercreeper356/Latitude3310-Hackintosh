# Latitude3310-Hackintosh
Fixes for specific issues when installing macOS on a Dell Latitude 3310

#Dell Latitude 3310: Hackintosh quirks

In order to install hackintosh, most of the hardware will work by 
following the Dortania guide.
There are 5 main quirks with this machine, of which I've figured out for 
you and documented here.
Thanks to the /r/Hackintosh Paradise Discord for their help. The Trackpad 
and GPU fixes wouldn't have been
possible without them.

The MacBookPro15,2 SMBIOS is optimal here, especially for GPU patching.
boot-args: debug=0x100 keepsyms=1 alcid=11
 igfxagdc=0 igfxblt msgbuf=1048576

##1. NVMe:
Even with NVMeFix kext, make sure the SATA controller is set to AHCI mode 
and not RAID mode. Even though there
are no SATA devices installed, this still effects the NVMe SSD.

##2. Backlight:
You may have to experiment with different solutions, but for me these 
options worked.

Add "igfxblt" to boot-args

In config.plist:
DeviceProperties > Add > PciRoot(0x0)/Pci(0x2,0x0)
Set enable-backlight-registers-fix (Data) to 01000000

##3. Trackpad:
This is not an I2C patching guide. If you don't understand, you should 
look over trackpad patching with VoodooI2C. These values and
configuration are used for this machine.

Using VoodooI2C, the trackpad should function at least, but with use you 
may notice that sensitivity setting,
multi touch, gestures, and palm rejection do not work properly, making it 
difficult to use. This machine's
trackpad requires extra work to patch.

https://github.com/5T33Z0/OC-Little-Translated/tree/main/05_Laptop-specific_Patches/Trackpad_Patches/I2C_TrackPad_Patches#other-i2c-hotpatches

In config.plist:
Kernel > Add
Disable VoodooPS2Mouse.kext and VoodooPS2Trackpad.kext

Add in SSDT-USTP.dsl
in config.plist rename USTP to XSTP rule
add SSDT-I2C-SPED (even if SSCN and FMCN are already present in DSDT)
snapshot OC in propertree

--vi2c-force-polling was NOT required in the boot args

##4. GPU Display Outputs:
This is not an iGPU patching guide. These are specific fixes which have 
worked on this machine.
If you don't undersand, follow guides for Whatevergreen iGPU, Framebuffer, 
and BusID patching, using this information where applicable.


Use disable-agdc by settings the boot argument "igfxagdc=0"
Important!

In config.plist:
DeviceProperties > Add > PciRoot(0x0)/Pci(0x2,0x0)
Set AAPL,ig-platform-id to 00009B3E
Set device-id to A53E0000

You now need to patch the connector types and BusID's. This can all be 
done by setting these values:
framebuffer-patch-enable > 01000000
framebuffer-con1-enable > 01000000
framebuffer-con1-alldata > 01011200 00080000 87010000
framebuffer-con2-enable > 01000000
framebuffer-con2-alldata > 02021200 00040000 87010000

Your DeviceProperties section should look like this when you are done:
![GpuPatch](https://github.com/silvercreeper356/Latitude3310-Hackintosh/assets/76752846/62a41fd8-1b5d-4093-9514-49d53c081016)


If everything is done right, you should be able to use displays with HDMI and DisplayPort over the Type-C port.


##5. FileVault:
FileVault can be enabled by following Dortania post-install guide for 
FileVault, and then can be enabled from within
macOS settings. The only issue is that this causes the time for it to ask 
for your password after power on
to be very slow, a minute or 2 atleast.
This is caused by Dell trackpads, to speed it up you can either move 
around on the trackpad or change these values:

Config.plist UEFI > AppleInpput

PointerPollMask set to 1
PointerPollMin set to 50
PointerPollMax set to 100
PointerSpeedMul set to 3

