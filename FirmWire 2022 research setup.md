sources: https://github.com/FirmWire/ndss22_experiments
https://firmwire.github.io/docs/interactive.html

1. clone the above repo, in same folder as FirmWire
2. exec the following command to execute in container:

```
sudo docker run --rm -it -v $(pwd):/baseband -v $PWD/FirmWire:/firmwire -p 3333:3333 -p 5555:5555 -p 3334:3334 firmwire
```

Here, the port 3333, 5555, and 3334 are exposed to host, useful for remote (host to container) gdb debugging
 the current folder is mounted at /baseband of the container, and FirmWire at /firmwire.
 The ndss22_experiements is at /baseband/ndss22_experiments

Export these to env variables in container to run vulnerability reproduction script

```
export FIRMWIRE_ROOT=/firmwire
export EXPERIMENT_ROOT=/baseband/ndss22_experiments/
```

# Recreating Vulnerabilities

The folder `ndss22_experiments/V-D_vulnerabilities` contains python script repro.py used to reproduce the vulnerabilities, and it's `crashes/` folder has the raw payload which cause the crash.

```
usage: repro.py [-h] [--firmware-override FIRMWARE_OVERRIDE]
                {CC1,CC2,RRC1,RRC2,RRC3,RRC4,SM}
repro.py: error: the following arguments are required: bugname
```

CC1, CC2 - Vulnerabilities in GSM CC Stack
RRC1..4 - Vulnerabilities in LTE RRC Stack 

Example to recreate CC1

```
python3 repro.py CC1
```

# GDB Connection

Example to recreate crashes with gdb connection support

```
./firmwire.py --fuzz-triage gsm_cc --fuzz-input /baseband/ndss22_experiments/V-D_vulnerabilities/crashes/GSM_CC_Bearer_Crash_MEM_GUARD_G973F_ASG8.bin modem.bin -s 5555
```

Here, -s 5555 opens port 5555 for remote gdb connections.
To connect with gdb, use `gdb-multiarch`, not plain `gdb`

```
$ gdb-multiarch
```

```
(gdb) 
(gdb) target remote :5555
Remote debugging using :5555
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x406cd35a in ?? ()
(gdb) info reg
r0             0x42f6ccc8          1123470536
r1             0x1a66              6758
r2             0x40771418          1081545752
r3             0x13d               317
r4             0x40444934          1078217012
r5             0x41699d00          1097440512
r6             0xbd                189
r7             0x13d               317
r8             0x40771418          1081545752
r9             0x402b6100          1076584704
r10            0x1a48              6728
r11            0x1a84              6788
r12            0x0                 0
sp             0x42f6ccc0          0x42f6ccc0
lr             0x40fc2d79          1090268537
pc             0x406cd35a          0x406cd35a
cpsr           0x60000033          1610612787
(gdb) 

```

Connect to existing container [[FirmWire Setup#Connect to existing container]]