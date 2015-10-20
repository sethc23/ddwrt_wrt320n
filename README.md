
# Open-Source DD-WRT Install for Linksys WRT320N

#### [Specs & Wiki](http://dd-wrt.com/wiki/index.php/Linksys_WRT320N_v1.0)
#### [Build Variation Table](http://dd-wrt.com/wiki/index.php/What_is_DD-WRT%3F#V24_pre_sp2_K26)
- openVPN build seems appealing but doesn't work with JFFS
- JFFS allows storage space for things like startup scripts configuring iptables

### To Enable SSH
- login via telnet (`telnet $IP`) to establish user and pw
- login via ssh BUT NO SSH-KEY !!
- login via ssh with key


### Add/Enable Optware in DDWRT

```bash
scp -r ./initial_setup/1_install_optware root@10.0.0.50:/jffs/tmp/
ssh root@$IP '
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
```


### Note:

This router, and it's newer counterpart (E2000) use the Broadcom 4717 CPU. Broadcom uses a default clock of 300MHz for the CPU, while Linksys has overclocked the CPU to 354MHz. This not only causes excessive heat which reduces routing performance, but is also responsible for many of the Wireless errors reported in the forum. It is suggested that you set the CPU clock to 300MHz until the developers have acknowledge the problem, and newer builds are released to fix it.

```bash
ssh root@$IP 'nvram set clkfreq=300,150,75; nvram commit; reboot'
```