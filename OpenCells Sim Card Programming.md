References:
https://open-cells.com/index.php/uicc-tutorial/
https://open-cells.com/index.php/uiccsim-programing/
Tool source: https://open-cells.com/d5138782a8739209ec5760865b1e53b0/uicc-v3.3.tgz
```
sudo ./program_uicc --adm 12345678 --imsi 208920100001101 --isdn 00000001 --acc 0001 --key 6874736969202073796d4b2079650a73 --opc 504f20634f6320504f50206363500a4f -spn "OpenCells01" --authenticate --noreadafter
```

```
coe@coe:~/baseband/uicc-v3.3$ sudo ./program_uicc

Existing values in USIM
ICCID: 89330061100000000198
WARNING: iccid luhn encoding of last digit not done 
USIM IMSI: 208920100001101
PLMN selector: : 0x02f8297c
Operator Control PLMN selector: : 0x02f8297c
Home PLMN selector: : 0x02f8297c
USIM MSISDN: 00000001
USIM Service Provider Name: OpenCells01

No ADM code of 8 figures, can't program the UICC

```

```
add_user imsi=208920100001101 imei=354622100893158 k=6874736969202073796d4b2079650a73
```

```
sudo ./program_uicc --adm 12345678 --imsi 208920100001102 --isdn 00000002 --acc 0001 --key 6874736969202073796d4b2079650a74 --opc 504f20634f6320504f50206363500a4f -spn "CoE SIM" --authenticate --noreadafter

Existing values in USIM
ICCID: 89330061100000000197
WARNING: iccid luhn encoding of last digit not done 
USIM IMSI: 208920100001197
PLMN selector: : 0x02f8297c
Operator Control PLMN selector: : 0x02f8297c
Home PLMN selector: : 0x02f8297c
USIM MSISDN: 00000197
USIM Service Provider Name: OpenCells197

Setting new values
Succeeded to authentify with SQN: 64
set HSS SQN value as: 96

```

NAT:
```
 1850  sudo iptables -t nat -L POSTROUTING -v -n
 1852  sudo iptables -L FORWARD -v -n
 1853  sudo iptables -A FORWARD -i eno2np1 -o docker0 -m state --state RELATED,ESTABLISHED -j ACCEPT
 1854  sudo iptables -t nat -L POSTROUTING -v 
 1855  iptables list
 1856  cat routehistory.txt | grep -e "iptables"
 1857  sudo iptables -t nat -L POSTROUTING -n -v
 1858  sudo iptables -L FORWARD -v
 1859  sudo iptables -A FORWARD -i eno2np1 -o srs_spgw_sgi -m state --state RELATED,ESTABLISHED -j ACCEPT
 1861  cat routehistory.txt | grep -e "iptables"
 1862  sudo iptables -t nat -L POSTROUTING -n -v
 1864  sudo iptables -t nat -A POSTROUTING -s 172.16.0.1/24 -o eno2np1 -j MASQUERADE
 1865  sudo iptables -A FORWARD -i srs_spgw_sgi -o eno2np1 -j ACCEPT
 1866  sudo iptables -A FORWARD -i eno2np1 -o srs_spgw_sgi -m state --state RELATED,ESTABLISHED -j ACCEPT
 2003  history | grep iptables

```

```
coe@coe:~/baseband$ sudo iptables -t nat -L POSTROUTING -v -n
Chain POSTROUTING (policy ACCEPT 7923 packets, 1586K bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    32 MASQUERADE  0    --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
  934  198K MASQUERADE  0    --  *      eno2np1  0.0.0.0/0            0.0.0.0/0           
    0     0 MASQUERADE  0    --  *      eno2np1  172.16.0.0/24        0.0.0.0/0           
coe@coe:~/baseband$ sudo iptables -L FORWARD -v -n
Chain FORWARD (policy DROP 234 packets, 61989 bytes)
 pkts bytes target     prot opt in     out     source               destination         
23927   19M DOCKER-USER  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
23927   19M DOCKER-FORWARD  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
14462   17M ACCEPT     0    --  eno2np1 srs_spgw_sgi  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
 9231 1787K ACCEPT     0    --  srs_spgw_sgi eno2np1  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     0    --  eno2np1 srs_spgw_sgi  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
coe@coe:~/baseband$ 

```

```
sudo srsepc_if_masq.sh eno2np1

sudo iptables -t nat -A POSTROUTING -s 172.16.0.1/24 -o eno2np1 -j MASQUERADE


sudo iptables -A FORWARD -i srs_spgw_sgi -o eno2np1 -j ACCEPT

sudo iptables -A FORWARD -i eno2np1 -o srs_spgw_sgi -m state --state RELATED,ESTABLISHED -j ACCEPT

```

```
  581  sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o eno2np1 -j MASQUERADE
  582  sudo iptables -A FORWARD -i ogstun -o eno2np1 -j ACCEPT
  583  sudo iptables -A FORWARD -i eno2np1 -o ogstun -m state --state RELATED,ESTABLISHED -j ACCEPT
```


