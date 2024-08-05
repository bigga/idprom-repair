# idprom-repair
SunBlade100/150 IDPROM randomizer (Linux)
(Tested on SunBlade 150)

# Sun EEPROM (nvram): Alternative method to restore/change Ethernet address and Host ID

You probably know that when you type banner at the PROM ok prompt, you see the system’s serial number, hostid and MAC address:
```
ok banner 
Sun Ultra 60 UPA/PCI (UltraSPARC-II 296MHz), Keyboard Present 
OpenBoot 3.11, 512 MB memory installed, 
Serial #10573911. Ethernet address 8:0:20:a1:58:57, Host ID: 80a15857 
```
Note that the serial number is the last 3 bytes of the HostID (a15857), converted to decimal.
This content is stored in a NVRAM chip (EEPROM) in the Sun workstation. The
contents of the NVRAM chip can become corrupted for a variety of reasons, most commonly, failure of the embedded battery.
The battery embedded in the NVRAM chip keeps the clock running when the machine is off and also maintains important system configuration information. 
This happens when turn on your Sun workstation and get output which looks something like:
```
Invalid format type in NVRAM
The IDPROM contents are invalid
ERROR: missing or invalid ID prom
Requesting Internet address for 0:0:0:0:0:0
```

Usually, when the NVRAM gets corrupted in this way, this is a symptom that the battery embedded in the
NVRAM chip has run out and you need to replace the chip. 

But you should try reprogramming the NVRAM chip with a hostid and ethernet address. 

This is what we will talk about.

Note: be sure to backup the original hostid and ethernet address values before modification. 
If your Sun EEPROM nvram is invalid, try to find original values (look stikers in your workstation or over nvram) and use this approarch.

This method is applicable to some Sun series, like Sun Blade100, Sun Blade150 (tested in a Sun Blade 150). 
Pay attention to small differences in the screen display and this guide, like the EEPROM path and virtual address that may be different, don't completely copy.


I will use my Blade 150 as an example, the original Host ID: 830f903b, Ethernet address:00:14:4f:0f:90:3b; will be restored. 

 
 1. Enter into OpenBoot prompt
 
 `Stop+A
 ok`
 
 2. Browse the device tree to find the full path to the EEPROM
 
 `ok show-devs` 
 
 3. Change to EEPROM full path:
 
 `ok cd /pci@1f,0/ebus@c/eeprom@1,0`
 
 Note that the EEPROM path to the EEPROM version of the output of different name, please refer to the screen display shall prevail. 
 
 4. List the name and values os the node's properties to find the value of device address
 
 `ok .properties`
 
 5. Use the addres value, in my case fff5a000
 
 `ok fff5a000 >physical`
 
 6. Displays the content of data stack automatically before each ok prompt
 
 `ok showstack`
 
 7. Maps a region of physical device address and return the allocated virtual address
 
 `ok 2000 memmap`
 
 8. Offset
 
 `ok 1fd8 +`
 
 9. Display 30 bytes of memory starting at current addr
 
 `ok 30 dump`
 
 Look at the output of the drawings, starting from fff51fd8 you mean as follows: 
```
 Byte address map
   0 fff51fd8:   Always 01 - Format/version number.
   1 fff51fd9:   1st hostid[0] byte / machine type [ 83 in Sun Blade, 80 in Sun Ultra,etc.]
 2-7 fff51fda~f: Ethernet (MAC) address 
 8-b fff51fe0~3: The date of manufacture, often all zeroes, is not a real date
   c fff51fe4:   2nd hostid[1] byte
   d fff51fe5    3rd hostid[2] byte
   e fff51fe6:   4th hostid[3] byte 
   f fff51fe7:   IDPROM checksum - bitwise xor of bytes 0-e (ie. xor of values from fff51fd8 to fff51fe6)
```
To restore the EEPROM values, we will write all these bytes and calculate the checksum. Remember in this example we will use Host ID: 83188869, Ethernet address:0:3:ba:18:88:69.

10. Write version (01) and hostid[0] byte/machine type (83)

```
ok 01 fff51fd8 c!
ok 83 fff51fd9 c!
```

11. Write ethernet address:00:14:4f:0f:90:3b

```
ok 00 fff51fda c!
ok 14 fff51fdb c!
ok 4f fff51fdc c!
ok 0f fff51fdd c!
ok 90 fff51fde c!
ok 3b fff51fdf c!
```

12. Write zeros (if not already) for the production date

```
ok 00 fff51fe0 c!
ok 00 fff51fe1 c!
ok 00 fff51fe2 c!
ok 00 fff51fe3 c!
```

13. Write hostid[1~3] bytes: 0f903b

```
ok 0f fff51fe4 c!
ok 90 fff51fe5 c!
ok 3b fff51fe6 c!
``` 

14. Calculate XOR-checksum for the values starting from fff51fd8, until fff51fe6

```
ok 01 83 xor
82 ok 00 xor
82 ok 14 xor
96 ok 4f xor
d9 ok 0f xor
d6 ok 90 xor
46 ok 3b xor
7d ok 0f xor
72 ok 90 xor
e2 ok 3b xor
d9 ok
```

15. Write calculared checksum

```
ok d9 fff51fe7 c!
```

16 Display Banner and reboot

```
ok banner
ok reset-all
```
