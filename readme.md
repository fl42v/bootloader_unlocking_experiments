# bootloader_unlocking_experiments
Trying to unlock phones' bootloaders without having a working display to manually check the oem unlocking switch

# Oneplus 5T

## Latest OxygenOS

> Note: The method described [here](https://alephsecurity.com/2018/01/22/qualcomm-edl-2/) (and, subsequently, [here](https://github.com/Giovix92/EDLUnlock)) doesn't work (devinfo seems to remain unchanged up to matching md5-s no matter the status of the bootloader)

### Method

> Note: Opposite to the way with patching `devinfo`, this procedure will still wipe the userdata. The only upside compared to the original way is that it's no longer necessary to have a working display at all.

The state of the oem unlocking switch is located in the `config` partition (diff between xxd-ed locked and unlocked config dumps):
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

Changing the `0x7ffff` byte did the trick without modifying the header at all.
On my device I was able to flash the whole partition using `edl w config config_patched.img`, and then `fastboot oem unlock` worked.

### How to (linux)

0. install [edl](https://github.com/bkerler/edl) and android platform tools
1. boot the phone in edl mode by powering it off then connecting it to the pc while pressing the volume up button (NO power button so far, otherwise it'll go into fastoot mode!)
2. backup the original `config` partition via `edl r config config_orig.img`
3. flip the last byte in the dumped image by your preferred method of choice (e.g. via hexcurse: `hexcurse config_orig.img` -> ctrl+g -> type 7ffff -> enter -> 01 -> ctrl q -> y -> type config_patched.img -> enter)
4. write the patched image back to the device: `edl w config config_patched.img`
5. reboot (disconnect the phone from the pc, long press the power button 'til the led stops glowing yellow)
6. go into fastboot (power + volume up while the phone is DISCONNECTED from the pc or power source) -> `fastboot oem unlock` -> volume down -> power button -> you're breathtaking

> Note: You may try to use my `config_patched.img` from the `./files/op5t` subdirectory. Before doing so make sure to backup the original partition (steps 0, 1 and 2) anyway in case it bricks stuff.

> Note: Also there's `config_unlocked.img` from a previous iteration where I made a mistake. Flashing it also worked.

### Questions:
* is it possible to go with patching `devinfo` on earlier versions of OxygenOS?

### Contributing
If you have a working device supported by edl that you don't currently use or can afford to format, you can help by determining if it's possible to do this on said device. The algorithm is as follows:

1. restore your device to factory ROM (there should be an article on xda or something; I restored mine with edl), then either update Android to the latest version the manufacturer provides or make note of which version you have;
2. lock the bootloader and uncheck the oem unlocking switch in the developer opitons;
3. boot into edl (the exact method differs from model to model but is easily duckduckgo-able) mode and dump the following partitions: CONFIG,FRP,PDB ([this](https://xdaforums.com/t/info-android-device-partitions-and-filesystems.3586565/) thread on xda suggests those are where the state of the switch is stored). For example, `edl r config config_locked`
4. reboot into the os (`edl reset` or by holding the power button) and allow oem unlocking;
5. once again boot into edl and dump those partitions (e.x. with "_unlocked" postfix);
6. `xxd` and `diff` the dumps pairwise:
   ```
   xxd config_locked xxd_config_locked
   xxd config_unlocked xxd_config_unlocked
   diff config_locked config_unlocked
   ```
7. if for some partition you see the results similar to what I encountered (i.e. one distinctly flipped bit), disable unlocking, follow the "how to" for your partition and bit and try unlocking the bootloader. In case it fauls, diff the xxd of modified partition against the _unlocked one to see if you've flipped the right bit (that's why my 1st attempt at unlocking this way failed; huge thanks to Mars, for pointing it out), otherwise `edl w` the unmodified dump back. THe other way would be to try flashing the _unlocked partitions one-by-one and testing them out -- at least this way we would know where exactly the state is stored;
8. make a pr or an issue with your findings.
