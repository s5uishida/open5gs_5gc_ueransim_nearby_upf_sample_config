# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select nearby UPF according to the connected gNodeB
This describes a very simple configuration that uses Open5GS and UERANSIM to select the nearby UPF according to the connected gNodeB.

---

<h2 id="toc">Table of Contents</h2>

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB1)](#changes_ran1)
    - [Changes in configuration files of RAN (gNodeB2)](#changes_ran2)
    - [Changes in configuration files of UE for Loc1 (IMSI-001010000000000)](#changes_ue_loc1)
    - [Changes in configuration files of UE for Loc2 (IMSI-001010000000000)](#changes_ue_loc2)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC C-Plane](#network_settings_cp)
  - [Network settings of Open5GS 5GC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS 5GC U-Plane2](#network_settings_up2)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run UERANSIM (gNodeBs)](#run_ran)
    - [Start gNodeB1 with TAC=1 in Loc1](#run_ran1)
    - [Start gNodeB2 with TAC=2 in Loc2](#run_ran2)
  - [Run UERANSIM (UE in Loc1)](#run_ue1)
    - [Start UE connected to gNodeB1 in Loc1](#con_ue1)
    - [Ping google.com going through DN=10.45.0.0/16 on Loc1](#ping_ue1)
  - [Run UERANSIM (UE in Loc2)](#run_ue2)
    - [Start UE connected to gNodeB2 in Loc2](#con_ue2)
    - [Ping google.com going through DN=10.46.0.0/16 on Loc2](#ping_ue2)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

The following minimum configuration was set as a condition.
- The pair of gNodeB and UPF exists in the same location.
- The UE connected to gNodeB connects to DN managed by UPF in the same location.

**In this example, TAC is matched to connect gNodeB and AMF, and AMF searches for SMF using TAC as part of the TAI parameter.**

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.4.7 - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.114/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.115/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB1) | 192.168.0.131/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM5 | UERANSIM RAN (gNodeB2) | 192.168.0.132/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM6 | UERANSIM UE | 192.168.0.133/24 | Ubuntu 20.04 | 1GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported TACs |
| --- | --- | --- | --- |
| AMF | 192.168.0.111 | 127.0.0.5 | 1, 2 |
| SMF1 | 192.168.0.112 | 127.0.0.4 | 1 |
| SMF2 | 192.168.0.113 | 127.0.0.24 | 2 |

gNodeB Information (other information is default) is as follows.  
| gNodeB # | Location # | TAC # | IP address |
| --- | --- | --- | --- |
| gNodeB1 | Loc1 | 1 | 192.168.0.131 |
| gNodeB2 | Loc2 | 2 | 192.168.0.132 |

Subscriber Information (other information is default) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE | IMSI | DNN | OP/OPc | gNodeB # |
| --- | --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc | gNodeB1 in Loc1 <br> gNodeB2 in Loc2|

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | Location # |  TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- | --- |
| 10.45.0.0/16 | Loc1 | ogstun | internet | uesimtun0 | U-Plane1 |
| 10.46.0.0/16 | Loc2 | ogstun | internet | uesimtun0 | U-Plane2 |

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.4.7 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2022-06-06 14:09:42.000000000 +0900
+++ amf.yaml    2022-06-06 14:14:58.000000000 +0900
@@ -229,23 +229,23 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     guami:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 901
-          mnc: 70
-        tac: 1
+          mcc: 001
+          mnc: 01
+        tac: [1, 2]
     plmn_support:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
     security:
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2022-06-06 14:09:42.000000000 +0900
+++ smf1.yaml   2022-06-06 14:18:48.000000000 +0900
@@ -381,17 +381,14 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.112
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.112
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -400,7 +397,17 @@
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf1.conf
+    info:
+      - s_nssai:
+          - sst: 1
+            dnn:
+              - internet
+        tai:
+          - plmn_id:
+              mcc: 001
+              mnc: 01
+            tac: 1
 
 #
 # nrf:
@@ -501,7 +508,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
+        dnn: internet
 
 #
 # parameter:
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2022-06-06 14:09:42.000000000 +0900
+++ smf2.yaml   2022-06-06 14:21:56.000000000 +0900
@@ -378,20 +378,17 @@
 
 smf:
     sbi:
-      - addr: 127.0.0.4
+      - addr: 127.0.0.24
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     gtpc:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 127.0.0.24
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -400,7 +397,17 @@
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf2.conf
+    info:
+      - s_nssai:
+          - sst: 1
+            dnn:
+              - internet
+        tai:
+          - plmn_id:
+              mcc: 001
+              mnc: 01
+            tac: 2
 
 #
 # nrf:
@@ -501,7 +508,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
+        dnn: internet
 
 #
 # parameter:
```
- `open5gs/install/etc/freeDiameter/smf1.conf`  
`smf1.conf` is equal to the original `smf.conf`.
- `open5gs/install/etc/freeDiameter/smf2.conf`
```diff
--- smf.conf.orig       2021-08-09 14:06:48.000000000 +0000
+++ smf2.conf   2021-08-09 16:01:40.000000000 +0000
@@ -79,7 +79,7 @@
 #ListenOn = "202.249.37.5";
 #ListenOn = "2001:200:903:2::202:1";
 #ListenOn = "fe80::21c:5ff:fe98:7d62%eth0";
-ListenOn = "127.0.0.4";
+ListenOn = "127.0.0.24";
 
 
 ##############################################################
```

<h3 id="changes_up1">Changes in configuration files of Open5GS 5GC U-Plane1</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2022-06-06 14:09:42.000000000 +0900
+++ upf.yaml    2022-06-06 14:29:06.000000000 +0900
@@ -166,12 +166,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
 
 #
 # smf:
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2022-06-06 14:09:42.000000000 +0900
+++ upf.yaml    2022-06-06 14:32:46.000000000 +0900
@@ -166,12 +166,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
+        dev: ogstun
 
 #
 # smf:
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran1">Changes in configuration files of RAN (gNodeB1)</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2021-04-19 21:30:50.000000000 +0000
+++ open5gs-gnb.yaml    2021-08-09 16:09:56.000000000 +0000
@@ -1,17 +1,17 @@
-mcc: '901'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<h4 id="changes_ran2">Changes in configuration files of RAN (gNodeB2)</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2021-04-19 21:30:50.000000000 +0000
+++ open5gs-gnb.yaml    2021-08-09 16:10:18.000000000 +0000
@@ -1,17 +1,17 @@
-mcc: '901'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
-tac: 1              # Tracking Area Code
+tac: 2              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.132   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.132   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.132    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<h4 id="changes_ue_loc1">Changes in configuration files of UE for Loc1 (IMSI-001010000000000)</h4>

- `UERANSIM/config/open5gs-ue-loc1.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-15 13:38:14.000000000 +0900
+++ open5gs-ue-loc1.yaml        2022-06-06 14:47:38.000000000 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
```

<h4 id="changes_ue_loc2">Changes in configuration files of UE for Loc2 (IMSI-001010000000000)</h4>

- `UERANSIM/config/open5gs-ue-loc2.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-15 13:38:14.000000000 +0900
+++ open5gs-ue-loc2.yaml        2022-06-06 14:48:32.000000000 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.132
 
 # UAC Access Identities Configuration
 uacAic:
```

<h2 id="network_settings">Network settings of Open5GS 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_cp">Network settings of Open5GS 5GC C-Plane</h3>

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.112/24 dev enp0s8
ip addr add 192.168.0.113/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<h3 id="network_settings_up1">Network settings of Open5GS 5GC U-Plane1</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

<h3 id="network_settings_up2">Network settings of Open5GS 5GC U-Plane2</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.46.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun -j MASQUERADE
```

<h2 id="build">Build Open5GS and UERANSIM</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.4.7 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

Note. Install MongoDB with package manager on Open5GS 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
```
# apt update
# apt install mongodb
# systemctl start mongodb
# systemctl enable mongodb
```
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 5
./install/bin/open5gs-smfd -c install/etc/open5gs/smf1.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf2.yaml &
./install/bin/open5gs-amfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<h3 id="run_up">Run Open5GS 5GC U-Plane1 & U-Plane2</h3>

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane1
```
./install/bin/open5gs-upfd &
```
- Open5GS 5GC U-Plane2
```
./install/bin/open5gs-upfd &
```
Then run `tcpdump` on one more terminal for each U-Plane.
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane2)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
```

<h3 id="run_ran">Run UERANSIM (gNodeBs)</h3>

Run each gNodeB with TAC=1 and TAC=2 in two locations.  
Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h4 id="run_ran1">Start gNodeB1 with TAC=1 in Loc1</h4>

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2022-06-07 00:01:28.141] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2022-06-07 00:01:28.143] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2022-06-07 00:01:28.144] [sctp] [debug] SCTP association setup ascId[4]
[2022-06-07 00:01:28.144] [ngap] [debug] Sending NG Setup Request
[2022-06-07 00:01:28.145] [ngap] [debug] NG Setup Response received
[2022-06-07 00:01:28.145] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
06/07 00:01:28.141: [amf] INFO: gNB-N2 accepted[192.168.0.131]:50968 in ng-path module (../src/amf/ngap-sctp.c:113)
06/07 00:01:28.141: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:619)
06/07 00:01:28.142: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:876)
06/07 00:01:28.142: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:658)
```

<h4 id="run_ran2">Start gNodeB2 with TAC=2 in Loc2</h4>

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2022-06-07 00:02:25.990] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2022-06-07 00:02:25.993] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2022-06-07 00:02:25.993] [sctp] [debug] SCTP association setup ascId[4]
[2022-06-07 00:02:25.993] [ngap] [debug] Sending NG Setup Request
[2022-06-07 00:02:25.994] [ngap] [debug] NG Setup Response received
[2022-06-07 00:02:25.994] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
06/07 00:02:25.991: [amf] INFO: gNB-N2 accepted[192.168.0.132]:59332 in ng-path module (../src/amf/ngap-sctp.c:113)
06/07 00:02:25.991: [amf] INFO: gNB-N2 accepted[192.168.0.132] in master_sm module (../src/amf/amf-sm.c:619)
06/07 00:02:25.991: [amf] INFO: [Added] Number of gNBs is now 2 (../src/amf/context.c:876)
06/07 00:02:25.991: [amf] INFO: gNB-N2[192.168.0.132] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:658)
```

<h3 id="run_ue1">Run UERANSIM (UE in Loc1)</h3>

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<h4 id="con_ue1">Start UE connected to gNodeB1 in Loc1</h4>

```
# ./nr-ue -c ../config/open5gs-ue-loc1.yaml 
UERANSIM v3.2.6
[2022-06-07 00:03:23.431] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-06-07 00:03:23.431] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-06-07 00:03:23.432] [nas] [info] Selected plmn[001/01]
[2022-06-07 00:03:23.432] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2022-06-07 00:03:23.432] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-06-07 00:03:23.433] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-06-07 00:03:23.433] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-06-07 00:03:23.434] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-07 00:03:23.434] [nas] [debug] Sending Initial Registration
[2022-06-07 00:03:23.434] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-06-07 00:03:23.435] [rrc] [debug] Sending RRC Setup Request
[2022-06-07 00:03:23.436] [rrc] [info] RRC connection established
[2022-06-07 00:03:23.436] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-06-07 00:03:23.436] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-06-07 00:03:23.446] [nas] [debug] Authentication Request received
[2022-06-07 00:03:23.449] [nas] [debug] Security Mode Command received
[2022-06-07 00:03:23.449] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-06-07 00:03:23.462] [nas] [debug] Registration accept received
[2022-06-07 00:03:23.463] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-06-07 00:03:23.463] [nas] [debug] Sending Registration Complete
[2022-06-07 00:03:23.463] [nas] [info] Initial Registration is successful
[2022-06-07 00:03:23.464] [nas] [debug] Sending PDU Session Establishment Request
[2022-06-07 00:03:23.464] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-07 00:03:23.674] [nas] [debug] Configuration Update Command received
[2022-06-07 00:03:23.693] [nas] [debug] PDU Session Establishment Accept received
[2022-06-07 00:03:23.697] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-06-07 00:03:23.721] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
06/07 00:03:23.408: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
06/07 00:03:23.409: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2072)
06/07 00:03:23.409: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:497)
06/07 00:03:23.410: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1398)
06/07 00:03:23.410: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1197)
06/07 00:03:23.411: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:134)
06/07 00:03:23.411: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:157)
06/07 00:03:23.411: [app] WARNING: Try to discover [AUSF] (../lib/sbi/path.c:110)
06/07 00:03:23.412: [amf] INFO: [6977149c-e5a9-41ec-8151-e337ca534306] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/07 00:03:23.413: [amf] INFO: [6977149c-e5a9-41ec-8151-e337ca534306] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/07 00:03:23.414: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/07 00:03:23.414: [ausf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF registered (../src/ausf/nnrf-handler.c:285)
06/07 00:03:23.415: [ausf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF Profile updated (../src/ausf/nnrf-handler.c:333)
06/07 00:03:23.421: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/07 00:03:23.422: [amf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/07 00:03:23.422: [amf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/07 00:03:23.426: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:110)
06/07 00:03:23.427: [amf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/07 00:03:23.427: [amf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/07 00:03:23.428: [app] WARNING: Try to discover [UDR] (../lib/sbi/path.c:110)
06/07 00:03:23.429: [pcf] INFO: [697c1b54-e5a9-41ec-bd80-79fc59629fbf] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:288)
06/07 00:03:23.429: [pcf] INFO: [697c1b54-e5a9-41ec-bd80-79fc59629fbf] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:350)
06/07 00:03:23.640: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1063)
06/07 00:03:23.642: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:389)
06/07 00:03:23.642: [gmm] INFO:     UTC [2022-06-06T15:03:23] Timezone[0]/DST[0] (../src/amf/gmm-build.c:502)
06/07 00:03:23.643: [gmm] INFO:     LOCAL [2022-06-07T00:03:23] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:507)
06/07 00:03:23.645: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2084)
06/07 00:03:23.646: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1068)
06/07 00:03:23.648: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:846)
06/07 00:03:23.649: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2868)
06/07 00:03:23.650: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/07 00:03:23.651: [smf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:03:23.651: [smf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/07 00:03:23.653: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:110)
06/07 00:03:23.654: [smf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:03:23.655: [smf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/07 00:03:23.656: [app] WARNING: Try to discover [BSF] (../lib/sbi/path.c:110)
06/07 00:03:23.657: [pcf] INFO: [6976bf24-e5a9-41ec-ace6-d932889486a4] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:288)
06/07 00:03:23.657: [pcf] INFO: [6976bf24-e5a9-41ec-ace6-d932889486a4] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:350)
06/07 00:03:23.658: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:492)
06/07 00:03:23.659: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
06/07 00:03:23.659: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:110)
06/07 00:03:23.660: [smf] INFO: [697ad4b0-e5a9-41ec-8312-53319a20c697] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:03:23.660: [smf] INFO: [697ad4b0-e5a9-41ec-8312-53319a20c697] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
```
The Open5GS U-Plane1 log when executed is as follows.
```
06/07 00:03:23.681: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:178)
06/07 00:03:23.681: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
06/07 00:03:23.681: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:397)
06/07 00:03:23.688: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
6: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::c69c:7869:841d:ebb5/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue1">Ping google.com going through DN=10.45.0.0/16 on Loc1</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.26.238) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.26.238: icmp_seq=1 ttl=61 time=25.6 ms
64 bytes from 172.217.26.238: icmp_seq=2 ttl=61 time=19.2 ms
64 bytes from 172.217.26.238: icmp_seq=3 ttl=61 time=19.6 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
00:05:42.875865 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 3, seq 1, length 64
00:05:42.898769 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 3, seq 1, length 64
00:05:43.877164 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 3, seq 2, length 64
00:05:43.894229 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 3, seq 2, length 64
00:05:44.878832 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 3, seq 3, length 64
00:05:44.896115 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 in the same Loc1 according to the connected gNodeB1 in Loc1.**

<h3 id="run_ue2">Run UERANSIM (UE in Loc2)</h3>

Then the UE disconnects from gNodeB1 and connects to gNodeB2 in Loc2.

<h4 id="con_ue2">Start UE connected to gNodeB2 in Loc2</h4>

```
# ./nr-ue -c ../config/open5gs-ue-loc2.yaml 
UERANSIM v3.2.6
[2022-06-07 00:06:56.988] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-06-07 00:06:56.989] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-06-07 00:06:56.990] [nas] [info] Selected plmn[001/01]
[2022-06-07 00:06:56.990] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2022-06-07 00:06:56.990] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-06-07 00:06:56.991] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-06-07 00:06:56.991] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-06-07 00:06:56.992] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-07 00:06:56.992] [nas] [debug] Sending Initial Registration
[2022-06-07 00:06:56.993] [rrc] [debug] Sending RRC Setup Request
[2022-06-07 00:06:56.993] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-06-07 00:06:56.994] [rrc] [info] RRC connection established
[2022-06-07 00:06:56.994] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-06-07 00:06:56.995] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-06-07 00:06:57.002] [nas] [debug] Authentication Request received
[2022-06-07 00:06:57.005] [nas] [debug] Security Mode Command received
[2022-06-07 00:06:57.005] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-06-07 00:06:57.014] [nas] [debug] Registration accept received
[2022-06-07 00:06:57.014] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-06-07 00:06:57.015] [nas] [debug] Sending Registration Complete
[2022-06-07 00:06:57.015] [nas] [info] Initial Registration is successful
[2022-06-07 00:06:57.015] [nas] [debug] Sending PDU Session Establishment Request
[2022-06-07 00:06:57.016] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-07 00:06:57.220] [nas] [debug] Configuration Update Command received
[2022-06-07 00:06:57.239] [nas] [debug] PDU Session Establishment Accept received
[2022-06-07 00:06:57.240] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-06-07 00:06:57.259] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
06/07 00:06:56.963: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
06/07 00:06:56.963: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2072)
06/07 00:06:56.963: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[2] CellID[0x10] (../src/amf/ngap-handler.c:497)
06/07 00:06:56.963: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] known UE by SUCI (../src/amf/context.c:1396)
06/07 00:06:56.963: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:134)
06/07 00:06:56.963: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:157)
06/07 00:06:56.964: [amf] INFO: UE Context Release [Action:1] (../src/amf/ngap-handler.c:1405)
06/07 00:06:56.964: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/ngap-handler.c:1406)
06/07 00:06:56.965: [amf] INFO: [Removed] Number of gNB-UEs is now 1 (../src/amf/context.c:2078)
06/07 00:06:56.966: [smf] INFO: Removed Session: UE IMSI:[imsi-001010000000000] DNN:[internet:1] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1540)
06/07 00:06:56.966: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:2874)
06/07 00:06:56.967: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:901)
06/07 00:06:56.967: [amf] INFO: [imsi-001010000000000:1] Release SM context [204] (../src/amf/amf-sm.c:451)
06/07 00:06:56.967: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2090)
06/07 00:06:56.979: [pcf] WARNING: NF EndPoint updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:93)
06/07 00:06:57.184: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1063)
06/07 00:06:57.185: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:389)
06/07 00:06:57.186: [gmm] INFO:     UTC [2022-06-06T15:06:57] Timezone[0]/DST[0] (../src/amf/gmm-build.c:502)
06/07 00:06:57.187: [gmm] INFO:     LOCAL [2022-06-07T00:06:57] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:507)
06/07 00:06:57.188: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2084)
06/07 00:06:57.188: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1068)
06/07 00:06:57.190: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:846)
06/07 00:06:57.191: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2868)
06/07 00:06:57.191: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/07 00:06:57.193: [smf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:06:57.194: [smf] INFO: [6976d25c-e5a9-41ec-ad14-43376ecf8842] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/07 00:06:57.198: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:110)
06/07 00:06:57.199: [smf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:06:57.199: [smf] INFO: [697c6dca-e5a9-41ec-a62b-310b7c640deb] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/07 00:06:57.201: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:492)
06/07 00:06:57.202: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
06/07 00:06:57.203: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:110)
06/07 00:06:57.203: [smf] INFO: [697ad4b0-e5a9-41ec-8312-53319a20c697] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/07 00:06:57.204: [smf] INFO: [697ad4b0-e5a9-41ec-8312-53319a20c697] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
```
The Open5GS U-Plane2 log when executed is as follows.
```
06/07 00:06:57.216: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:178)
06/07 00:06:57.216: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
06/07 00:06:57.216: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:397)
06/07 00:06:57.221: [gtp] INFO: gtp_connect() [192.168.0.132]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::95ac:7201:6b2f:d9c9/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue2">Ping google.com going through DN=10.46.0.0/16 on Loc2</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.31.142) from 10.46.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.31.142: icmp_seq=1 ttl=61 time=20.6 ms
64 bytes from 172.217.31.142: icmp_seq=2 ttl=61 time=18.8 ms
64 bytes from 172.217.31.142: icmp_seq=3 ttl=61 time=18.6 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
00:08:49.720410 IP 10.46.0.2 > 172.217.31.142: ICMP echo request, id 4, seq 1, length 64
00:08:49.737952 IP 172.217.31.142 > 10.46.0.2: ICMP echo reply, id 4, seq 1, length 64
00:08:50.721785 IP 10.46.0.2 > 172.217.31.142: ICMP echo request, id 4, seq 2, length 64
00:08:50.738229 IP 172.217.31.142 > 10.46.0.2: ICMP echo reply, id 4, seq 2, length 64
00:08:51.723584 IP 10.46.0.2 > 172.217.31.142: ICMP echo request, id 4, seq 3, length 64
00:08:51.739664 IP 172.217.31.142 > 10.46.0.2: ICMP echo reply, id 4, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2022.06.07] Updated to Open5GS v2.4.7 and UERANSIM v3.2.6.
- [2021.08.17] Initial release.
