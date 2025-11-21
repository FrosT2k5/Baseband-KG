References:
MCC MNC Values reference: https://www.cyberyodha.org/2022/06/mcc-mnc-mobile-network-code-in-india.html | https://mcc-mnc.net/
LTE Radio bands & EARFCN Calculator: https://www.sqimway.com/lte_band.php
SrsRAN Installation: [[SrsRAN Installation]]
SrsRAN Default Configuration: https://github.com/srsran/srsRAN_4G/tree/master/srsenb (*.conf.example files)

SrsRAN Configuration directory: `/root/.config/srsran/`

Tested on version:
```
$ uhd_usrp_probe --version
4.8.0.0-266-g50967d13

$ srsenb --version
Active RF plugins: libsrsran_rf_uhd.so libsrsran_rf_zmq.so
Inactive RF plugins: 
---  Software Radio Systems LTE eNodeB  ---
Version 23.4.0
```
## EPC Configuration
EPC is the Evolved Packet Core, the core server that handles the LTE Network (packet, routing, user management, subscriber management, etc) and connects with one or more ENodeBs which act as the Network Tower which Mobile devices can connect to.
In SRSRan, the epc configuration is in `epc.conf`.

Following values were modified from defaults:
### 1. MCC, MNC Values, APN
```
[mme]
...
mcc = 404
mnc = 57
# jio - mcc mnc 405 874
# bsnl - 404 57
apn = srsapn
```
These values belong to BSNL Network, taken from the mcc references links, This MCC and MNC was tested and working in Pixel 7, Nothing phone and xiaomi/oppo devices.
Set the apn here, it needs to be added in the APN Section of mobile network settings to access the internet
### 2. Network Name
```
[mme]
...
full_net_name="coe lte network"
short_net_name="coe"
```
Used to set network name that will be displayed after the device is connected to the LTE Network
### 3. Packet Capture
```
[pcap]
enable   = true
filename = /tmp/epc.pcap
```
Captures packets and stores it in `/tmp/epc.pcap` file. Can be used and inspected in wireshark, using following configuration in wireshark:
```
Packets are captured to file in the compact format decoded by 
the Wireshark s1ap dissector and with DLT 150. 
To use the dissector, edit the preferences for DLT_USER to 
add an entry with DLT=150, Payload Protocol=s1ap.
```
### 4. Logging
```
[log]
all_level = debug
all_hex_limit = 32
filename = /tmp/epc.log

nas_level = debug
s1ap_level = debug
mme_gtpc_level = debug
spgw_gtpc_level = debug
gtpu_level = debug
spgw_level = debug
hss_level = debug
```
Enables logging, saves logs to /tmp/epc.log

## ENodeB Configuration
ENodeB is a network node/endpoint that conencts end user devices with the LTE Core Network (EPC), for our setup this is part which requires USRP, tested with USRP B200, USB 3.0 Connection (SrsRAN DID NOT work with USB 2.0 Connection)
### rr.conf:
```
cell_list =
(
  {
    // rf_port = 0;
    cell_id = 0x01;
    // tac = 0x0007;
    tac = 0x0007;
    pci = 1;
    // root_seq_idx = 204;
    // dl_earfcn = 1700; worked on poco
    // dl_earfcn = 3350, 1700, 1350, 400;
    dl_earfcn = 3350;
    tx_gain = 20.0
```
Here, we need to set dl_earfcn value according to the LTE Band and Frequency we choose to operate, the 3350 is default factory value that is embedded in rr.conf, this can also be overriden through enb.conf as described below.
Values tested and working: 
```
3350 for Samsung S10 (MCC, MNC of JIO)
1700 for Poco/Xiaomi Devices
3600 for Pixel 7, Nothing Phone
```

`tx_gain` can be increased to increase signal strength, note that USRP might not be able to handle gain values too high, can be overriden from enb.conf
### enb.conf
#### MCC MNC Values
```
[enb]
...
mcc = 404
mnc = 57
```
MCC MNC values should be same as the ones in EPC
#### Bandwidth
```
n_prb = 75 # Tested with pixel 7
# default is 50
```
n_prb is used to set the bandwidth of the network, 75 corresponds to bandwidth of 15MHz
#### EARFCN, Gain
```
[rf]
# 3600 for pixel
dl_earfcn = 3600
tx_gain = 75
rx_gain = 30
```
earfcn value is used to derive the LTE Band and Uplink/Downlink frequency, use the earfcn calculator from reference to derive this value of the desired LTE Band, Note that this value shouldnt overlap with the existing LTE networks in the area.
Use `*#*#4636#*#*` to figure out earfcn values of available networks in Android devices

```
[rf]
...
device_args=master_clock_rate=53.76e6
```
device arguments, passed to uhd drivers. Used to set master_clock_rate, etc.
`master_clock_rate=15.36e6` tested and working for pixel.
#### Packet Capture
```
[pcap]
enable = true
filename = /tmp/enb_mac.pcap
nr_filename = /tmp/enb_mac_nr.pcap
s1ap_enable = true
s1ap_filename = /tmp/enb_s1ap.pcap
```
Capture ENodeB MAC Layer traffic and S1AP control plane protocol packets (ENodeB <-> EPC Packets)

## Adding Subscribers
Subscribers are added in `user_db.csv`
```
#                                                                                
# .csv to store UE's information in HSS                                          
# Kept in the following format: "Name,Auth,IMSI,Key,OP_Type,OP/OPc,AMF,SQN,QCI,IP_alloc"  
#                                                                                
# Name:     Human readable name to help distinguish UE's. Ignored by the HSS     
# Auth:     Authentication algorithm used by the UE. Valid algorithms are XOR    
#           (xor) and MILENAGE (mil)                                             
# IMSI:     UE's IMSI value                                                      
# Key:      UE's key, where other keys are derived from. Stored in hexadecimal   
# OP_Type:  Operator's code type, either OP or OPc                               
# OP/OPc:   Operator Code/Cyphered Operator Code, stored in hexadecimal          
# AMF:      Authentication management field, stored in hexadecimal               
# SQN:      UE's Sequence number for freshness of the authentication             
# QCI:      QoS Class Identifier for the UE's default bearer.                    
# IP_alloc: IP allocation stratagy for the SPGW.                                 
#           With 'dynamic' the SPGW will automatically allocate IPs            
#           With a valid IPv4 (e.g. '172.16.0.2') the UE will have a statically assigned IP.
#                                                                                
# Note: Lines starting by '#' are ignored and will be overwritten    
```
For our OpenCells SIM,
Auth: mil
Other values should match the ones the SIM Card is configured with, see [[OpenCells Sim Card Programming]]
Values for 2 SIM Cards I've programmed:
```
# "Name,Auth,IMSI,Key,OP_Type,OP/OPc,AMF,SQN,QCI,IP_alloc"  
op1,mil,208920100001101,6874736969202073796d4b2079650a73,opc,504f20634f6320504f50206363500a4f,9000,000000000852,9,dynamic
op2,mil,208920100001102,6874736969202073796d4b2079650a74,opc,504f20634f6320504f50206363500a4f,9000,0000000006e7,9,dynamic
```

## Execute EPC and ENodeB
### Execute EPC First:
```
sudo srsepc 
```
### In new terminal, execute ENB
```
sudo srsenb
```
If configured correctly, the srsepc should receive S1 connection request from the srsenb, and srsenb should successfully connect with EPC. The Rx/Tx both lights should light up in the USRP, connect antennas to both of the ports (Rx/Tx) of the USRP and they should be perpendicular to each other

### NAT Configuration
To access the internet from the network, we need to configure NAT using `iptables`
1. figure out default interface for internet
```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    600    0        0 eno2np1
192.168.29.0    0.0.0.0         255.255.255.0   U     600    0        0 wlp0s20f3

```
here, the default interface is `enp0s1`
2. Use SrsRAN helper script to enable NAT
`sudo srsepc_if_masq.sh eno2np1`
3. Run following commands to enable IP Forwarding
```
sudo iptables -t nat -A POSTROUTING -s 172.16.0.1/24 -o eno2np1 -j MASQUERADE
sudo iptables -A FORWARD -i srs_spgw_sgi -o eno2np1 -j ACCEPT
sudo iptables -A FORWARD -i eno2np1 -o srs_spgw_sgi -m state --state RELATED,ESTABLISHED -j ACCEPT
```
`srs_spgw_sgi` is the interface created by SrsEPC

## Test on Phone
If everything is configured and working correctly on the device,
The network should be available in "Select Network Manually" section of mobile networks in settings
Connecting to the network should be successful and Enabling Data Roaming should allow access to internet via LTE Network.
We need to create new APN with name from the epc.conf to access the internet from the device. 