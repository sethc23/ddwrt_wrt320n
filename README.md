
# Open-Source DD-WRT Install for Linksys WRT320N


[Specs & Wiki](http://dd-wrt.com/wiki/index.php/Linksys_WRT320N_v1.0) | [Builds](ftp://ftp.dd-wrt.com/others/eko/V24-K26) | [Build Variation Table](http://dd-wrt.com/wiki/index.php/What_is_DD-WRT%3F#V24_pre_sp2_K26)
---


- openVPN build seems appealing but doesn't work with JFFS
- JFFS allows storage space for things like startup scripts configuring iptables



- - -
- - -

Benefit: Significantly reduces networking complexity and debugging.
How: use linux tools to test and debug at real network endpoints
Setup:  Using a samba share to make socat and other useful tools available.

- - -
- - -


#### Initial Flash Files:

- dd-wrt.v24-14896_NEWD-2_K2.6_mini.bin

	- Good enough for setting up a basic firewall with iptables and doing some analysis with ulog.


#### Flash Upgrades:

- dd-wrt.v24-18946_NEWD-2_K2.6_big.bin


    NOTE: a very specific order of events is necessary when setting up a Repeater (and may apply to other configurations)
    
    	A configuration will fail and/or break if the lan settings are changed after wireless settings.




**NOTE:**

- 21061 builds listed on the wiki were not intended for the 320n, i.e., the GUI states the router model is WRT54G, and:

    - SSH fails

        - telnet worked
        - on first try, without any ssh keys:
            `debug1: expecting SSH2_MSG_KEXDH_REPLY`
        - same error even after applying settings and rebooting
        - same error even after `erase nvram; reboot`

## Configuration

#### Add/Enable Optware in DDWRT

```bash
scp -r ./initial_setup/* wrt:/jffs/tmp/; \
ssh wrt '
    cd /jffs/tmp
    ./1_install_optware
    rm 1_install_optware
    mkdir -p /jffs/var/log/
    ipkg-opt list_installed
    df -h'
```

### Copy Configurations

```bash
scp -r ./etc root@$IP:/jffs/
mount -o bind /jffs/opt /opt; export PATH=/opt/bin:/opt/sbin:$PATH
```




### Performance:

This router, and it's newer counterpart (E2000) use the Broadcom 4717 CPU. Broadcom uses a default clock of 300MHz for the CPU, while Linksys has overclocked the CPU to 354MHz. This not only causes excessive heat which reduces routing performance, but is also responsible for many of the Wireless errors reported in the forum. It is suggested that you set the CPU clock to 300MHz until the developers have acknowledge the problem, and newer builds are released to fix it.

```bash
ssh root@$IP 'nvram set clkfreq=300,150,75; nvram commit; reboot'
```

## ISSUES:

### Hard Reset (to firmware defaults):

1. Use telnet or ssh:
```bash
erase nvram
reboot
```

2. Web GUI:  
		Administration -> Factory Defaults

3. While with power and holding reset for the entire time: 

        wait 30 seconds
        disconnect power for 30 seconds
        power up again for 30 seconds.

4. Assuming build 13493 or better
        a) Unplug router
        b) Press and hold the WPS button on the front of the unit.
        c) Plug power back in with button still pressed, hold down button for 10-12 seconds.
        d) Release button after 10-12 seconds. Wait 2 minutes, then access webgui at 192.168.1.1
        

#### Others Builds:

###### dd-wrt.v24-14896_NEWD-2_K2.6_mini_hotspot.bin

- repeater settings wouldn't work (though more cool down time may have helped)

###### dd-wrt.v24-14896_NEWD-2_K2.6_big.bin

- same as above

###### openVPN
- dd-wrt.v24-14896_NEWD-2_K2.6_openvpn.bin
- dd-wrt.v24-14471_NEWD-2_K2.6_openvpn.bin

	Had some issues initially but decided samba option was best.

#### Extra Steps if SSH issue:
1. login via telnet (`telnet $IP`) to establish user and pw
2. login via ssh BUT NO SSH-KEY !!
3. login via ssh with key