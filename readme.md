# bootloader_unlocking_experiments
Trying to unlock phones' bootloaders without having a working display to manually check the oem unlocking switch

# Oneplus 5T

## Latest OxygenOS

> Note: The method described [here](https://alephsecurity.com/2018/01/22/qualcomm-edl-2/) (and, subsequently, [here](https://github.com/Giovix92/EDLUnlock)) doesn't work (devinfo seems to remain unchanged up to matching md5-s no matter the status of the bootloader)

The state of the oem unlocking switch is located in the `config` partition:
```diff
1,2c1,2
< 00000000: 798e 3004 9720 d0c9 66a7 fb73 cc44 1794  y.0.. ..f..s.D..
< 00000010: 8690 3fa6 0fe3 a926 2372 8ce8 11c3 e942  ..?....&#r.....B
---
> 00000000: d6bf e388 1a54 b083 ce2a 0dad d2d6 a95d  .....T...*.....]
> 00000010: cf75 1e1f c618 f27c 62c7 f130 75a4 b31a  .u.....|b..0u...
32768c32768
< 0007fff0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
---
> 0007fff0: 0000 0000 0000 0000 0000 0000 0000 0001  ................
```

Changing the `0x7fff7` byte doesn't do the trick: the header may contain some kind of checksum (or that's the wrong bit to flip).
On my device I was able to flash the whole partition using `edl w config config_unlocked.img`, and then `fastboot oem unlock` worked.

Questions:
* if that's a checksum, what does it depend on (os verions, particular device, etc)?
* if the partition's contents vary device-to-device or version-to-version, how to calculate them outside of the device?
* is it possible to go with patching `devinfo` on earlier versions of OxygenOS?

> Note: `config_unlocked.img` is in the `./files/op5t` subdirectory. Before trying out make sure to `edl r config config_orig.img` in case it bricks stuff.
