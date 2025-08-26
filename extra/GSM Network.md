# **YatesBTS:**  
  
The USRP we have is USRP B200, the latest version of yatesbts only works with BladeRF, not with UHD drivers based USRPs:

  
in the following blog, by the developers of firmwire, they’ve an old fork of yatesbts that does support USRP/UHD Drivers:

[https://hernan.de/blog/creating-a-cellular-testbed-with-yatebts-and-srslte/](https://hernan.de/blog/creating-a-cellular-testbed-with-yatebts-and-srslte/)

Repo of yatesbts fork:

```
https://github.com/grant-h/YateBTS-USRP
```

We’ve followed the installation instruction for yates, as well as yatesbts from the above blog.
Yate Version that’s compatible with this YateBTS: 5.5.0,
commit 94298ebbf1294740e38b24f111f24777f5e9163d from https://github.com/yatevoip/yate


## Steps Performed:

Used ubuntu 18.04 docker container, supported according to blog
1. Cloned both the repos, 
2. configured and make installed
3. modified the yatesbts configuration in /usr/local/etc/yate/

Changes made to configuration:

in `ybts.conf`

```
[gsm]
Radio.Band=900
Radio.C0=62					; Should be between 0..124 for the 900 band
Identity.MCC=405			; MCC, MNC taken from Old network that used to operate
Identity.MNC=805			; in India 4-5 years ago, since the phone is that old

[transceiver]
Path=./transceiver-uhd		; The UHD transceiver path, compiled from YatesBTS-USRP repo

[tapping]
GSM=yes						; To capture GSM packets with wireshark
```

in `subscribers.conf`
```
[general]
country_code=91
regexp=^809					
; Regex expression that match the IMSI, IMSI 
; begins with 809 as per the SIM Card Info
```
## Execution

1.  Run yatesbts with command `yate` in privledged docker container so that it can access the usb devices
``` 
sudo docker run -it --privileged --device /dev/bus/usb -p 4444:80 -p 5038:5038 -p 30000:30000 -p 30001:30001 -p 30002:30002 -p 30003:30003 -v /lib/modules:/lib/modules -v /etc/network/if-up.d:/etc/network/if-up.d yate-container
```

```
root@93725f335145:/# yate 
Yate (16) is starting Tue Aug 26 12:58:45 2025
Loaded module ExtModule
Loaded module GSM - based on libgsm-1.0.10
Loaded module YSTUN
Loaded module RegexRoute
Loaded module ToneDetector
...
Starting transceiver
ALERT 125839699810112 12:58:55.4 TRXManager.cpp:603:getFactoryCalibration: READFACTORY failed with status 1
2025-08-26_12:58:55.461158 <mbts:WARN> TRXManager.cpp:603:getFactoryCalibration: READFACTORY failed with status 1
MBTS ready
```

2. Connect to control interface of yatesbts
```
$ telnet 0 5038

Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.
YATE 5.5.0-1 r (http://YATE.null.ro) ready on 93725f335145.
mbts help

Type "help" followed by the command name for help on that command.

alarms		audit		cellid		
chans		config		crashme		
devconfig	freqcorr	gprs		
help		load		noise		
notices		page		power		
radio		rawconfig	regperiod	
reload		rmconfig	rxgain		
sgsn		shutdown	stats		
sysinfo		trxfactory	txatten		
unconfig	uptime		version		
```

So, the yates is running correctly and the network shows up in the device's 
"Choose Network Operators" menu
as the "Aircel", according to MCC and MNC Values

3. Register to the network 
Choose the network operator from 'Choose Network Operator' Menu in Device with SIM Card
![[networkoperators.png|300]]
However, registration Fails with notification on phone
```
No Service
Selected Network (AIRCEL) is not available
```

Tried with different MCC and MNC Values of different operators, with different radio bands and C0/ARFCN Values

In the yate control telnet connection, following commands can be used to check for registration attempts:
```
nib list registered
IMSI            MSISDN 
--------------- ---------------
nib list rejected
IMSI            No attempts register 
--------------- ---------------
```

No registration attempts were made, tapping in wireshark for "any" interface shows that GSM Packets for USRP -> Device are working but no packets are received back for Device -> USRP

# OpenBTS

Installed OpenBTS from a fork that claims to support latest UHD drivers and ubuntu 24.04:
https://github.com/PentHertz/OpenBTS

Also tried the original OpenBTS by RangeNetworks in a docker container

OpenBTS seems to receive some data when the created GSM network is created in the device. But still the registration fails.
Following messages are logged in OpenBTS Shell:
```
OpenBTS> ALERT 2127:2153 2025-08-26T09:23:46.0 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:23:47.1 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:23:48.2 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:23:55.0 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:23:56.2 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:23:57.5 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:24:15.3 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:24:16.3 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:24:17.5 Transceiver.cpp:414:pullRadioVector: Clipping detected on RACH input
ALERT 2127:2153 2025-08-26T09:24:24.8 Transceiver.cpp:414:pullRadioVector: 
```

the fork by PentHertz crashes time to time due to buffer overflows randomly.