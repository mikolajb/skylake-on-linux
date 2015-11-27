# ASRock Fatal1ty Z170 Gaming-ITX/ac and Intel Skylake processor on (arch) Linux

I wasted some time forcing this motherboard and Skylake processor to cooperate with my Linux. In order to spare you some time I publish all my findings.

## Graphics

Add the following kernel parameters:
`i915.enable_execlists=0 i915.ips=0`

- without `i915.enable_execlists=0` it just doesn't work
- without `i915.ips=0` it freezes

## ACPI AE_NOT_FOUND errors for GPE_L6F

There is an annoying ACPI error that ends with filling journal with logs, using processor and filling disk space. Updating BIOS doesn't help but there is a workaround for this - disable the interrupt as follows:

`echo "disable" > /sys/firmware/acpi/interrupts/gpe6F`

I also include a systemd unit that does it for you.

Links:
- https://bugzilla.kernel.org/show_bug.cgi?id=105491
- http://jhshi.me/2015/11/14/acpi-error-method-parseexecution-failed-_gpe_l6f/index.html

## WIFI

This is pretty straightforward:
https://aur.archlinux.org/packages/broadcom-wl-dkms/

There are similar solution for other distributions, e.g.:
https://help.ubuntu.com/community/WifiDocs/Driver/bcm43xx

## Bluetooth

It has Broadcom BCM94352, aka BCM4352, aka AzureWare2123 that (which is really annoying) partially works with Linux, you can scan for devices but you cannot pair nor connect. The solution is as follows:

Download windows drivers for Bluetooth from [asrock website](http://www.asrock.com/mb/Intel/Fatal1ty%20Z170%20Gaming-ITXac/?cat=Download&os=All)

Open the archive and do what `toz` explained [on archlinux forum](https://bbs.archlinux.org/viewtopic.php?id=197469):

- With `lsusb` we get a device ID:

```
...
Bus 001 Device 003: ID 13d3:3404 IMC Networks
...
```

- in `bcbtums-win8x64-brcm.inf` file, search for 3404:

```
[RAMUSB3404.CopyList]
bcbtums.sys
btwampfl.sys
BCM20702A1_001.002.014.1443.1479.hex
```

- it points to `BCM20702A1_001.002.014.1443.1479.hex` file, convert it to `hcd` as follows:

`hex2hcd BCM20702A1_001.002.014.1443.1479.hex`

- change file name to: `BCM20702A0-13d3-3404.hcd`
- copy it to `/lib/firmware/...`
- link it to `ln -rs BCM20702A1-13d3-3404.hcd`
