# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select nearby UPF according to the connected gNodeB
This describes a very simple configuration that uses Open5GS and UERANSIM to select the nearby UPF according to the connected gNodeB.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

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
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

The following minimum configuration was set as a condition.
- The pair of gNodeB and UPF exists in the same location.
- The UE connected to gNodeB connects to DN managed by UPF in the same location.

**In this example, TAC is matched to connect gNodeB and AMF, and AMF searches for SMF using TAC as part of the TAI parameter.**

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.0 (2024.03.24) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 | Ubuntu 22.04 | 2GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.114/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.115/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB1) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM RAN (gNodeB2) | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM6 | UERANSIM UE | 192.168.0.133/24 | Ubuntu 22.04 | 1GB | 10GB |

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

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ amf.yaml    2024-03-26 19:35:02.342023479 +0900
@@ -19,27 +19,27 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
-      tac: 1
+        mcc: 001
+        mnc: 01
+      tac: [1, 2]
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
   security:
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ nrf.yaml    2024-03-25 19:46:56.184797762 +0900
@@ -10,8 +10,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf1.yaml   2024-03-31 22:57:17.414819017 +0900
@@ -1,5 +1,5 @@
 logger:
-  file: /root/open5gs/install/var/log/open5gs/smf.log
+  file: /root/open5gs/install/var/log/open5gs/smf1.log
 #  level: info   # fatal|error|warn|info(default)|debug|trace
 
 global:
@@ -19,35 +19,41 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.112
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.0.114
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.112
   metrics:
     server:
       - address: 127.0.0.4
         port: 9090
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+  info:
+    - s_nssai:
+        - sst: 1
+          dnn:
+            - internet
+      tai:
+        - plmn_id:
+            mcc: 001
+            mnc: 01
+          tac: 1
 
 ################################################################################
 # SMF Info
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf2.yaml   2024-03-31 22:57:26.894698008 +0900
@@ -1,5 +1,5 @@
 logger:
-  file: /root/open5gs/install/var/log/open5gs/smf.log
+  file: /root/open5gs/install/var/log/open5gs/smf2.log
 #  level: info   # fatal|error|warn|info(default)|debug|trace
 
 global:
@@ -10,7 +10,7 @@
 smf:
   sbi:
     server:
-      - address: 127.0.0.4
+      - address: 127.0.0.24
         port: 7777
     client:
 #      nrf:
@@ -19,35 +19,41 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.113
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.0.115
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.113
   metrics:
     server:
-      - address: 127.0.0.4
+      - address: 127.0.0.24
         port: 9090
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.46.0.1/16
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+  info:
+    - s_nssai:
+        - sst: 1
+          dnn:
+            - internet
+      tai:
+        - plmn_id:
+            mcc: 001
+            mnc: 01
+          tac: 2
 
 ################################################################################
 # SMF Info
```

<a id="changes_up1"></a>

### Changes in configuration files of Open5GS 5GC U-Plane1

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-25 20:16:20.324142755 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.114
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.114
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_up2"></a>

### Changes in configuration files of Open5GS 5GC U-Plane2

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-25 20:18:50.813311682 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.46.0.1/16
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran1"></a>

#### Changes in configuration files of RAN (gNodeB1)

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:43.000000000 +0900
+++ open5gs-gnb.yaml    2023-01-12 22:31:00.200921228 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
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

<a id="changes_ran2"></a>

#### Changes in configuration files of RAN (gNodeB2)

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:43.000000000 +0900
+++ open5gs-gnb.yaml    2023-01-12 22:32:12.052740137 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
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

<a id="changes_ue_loc1"></a>

#### Changes in configuration files of UE for Loc1 (IMSI-001010000000000)

- `UERANSIM/config/open5gs-ue-loc1.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue-loc1.yaml        2024-03-26 20:05:20.085805718 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
```

<a id="changes_ue_loc2"></a>

#### Changes in configuration files of UE for Loc2 (IMSI-001010000000000)

- `UERANSIM/config/open5gs-ue-loc2.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue-loc2.yaml        2024-03-26 20:06:13.725206392 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.132
 
 # UAC Access Identities Configuration
 uacAic:
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC and UERANSIM UE / RAN

<a id="network_settings_cp"></a>

### Network settings of Open5GS 5GC C-Plane

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.112/24 dev enp0s8
ip addr add 192.168.0.113/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<a id="network_settings_up1"></a>

### Network settings of Open5GS 5GC U-Plane1

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

<a id="network_settings_up2"></a>

### Network settings of Open5GS 5GC U-Plane2

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

<a id="build"></a>

## Build Open5GS and UERANSIM

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<a id="run"></a>

## Run Open5GS 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd -c install/etc/open5gs/smf1.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf2.yaml &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<a id="run_up"></a>

### Run Open5GS 5GC U-Plane1 & U-Plane2

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

<a id="run_ran"></a>

### Run UERANSIM (gNodeBs)

Run each gNodeB with TAC=1 and TAC=2 in two locations.  
Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="run_ran1"></a>

#### Start gNodeB1 with TAC=1 in Loc1

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2024-03-26 20:31:13.709] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2024-03-26 20:31:13.719] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2024-03-26 20:31:13.719] [sctp] [debug] SCTP association setup ascId[5]
[2024-03-26 20:31:13.720] [ngap] [debug] Sending NG Setup Request
[2024-03-26 20:31:13.735] [ngap] [debug] NG Setup Response received
[2024-03-26 20:31:13.735] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/26 20:31:13.853: [amf] INFO: gNB-N2 accepted[192.168.0.131]:50048 in ng-path module (../src/amf/ngap-sctp.c:113)
03/26 20:31:13.853: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:754)
03/26 20:31:13.867: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1236)
03/26 20:31:13.868: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:793)
```

<a id="run_ran2"></a>

#### Start gNodeB2 with TAC=2 in Loc2

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2024-03-26 20:31:54.888] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2024-03-26 20:31:54.898] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2024-03-26 20:31:54.899] [sctp] [debug] SCTP association setup ascId[5]
[2024-03-26 20:31:54.899] [ngap] [debug] Sending NG Setup Request
[2024-03-26 20:31:54.914] [ngap] [debug] NG Setup Response received
[2024-03-26 20:31:54.914] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/26 20:31:54.899: [amf] INFO: gNB-N2 accepted[192.168.0.132]:52880 in ng-path module (../src/amf/ngap-sctp.c:113)
03/26 20:31:54.899: [amf] INFO: gNB-N2 accepted[192.168.0.132] in master_sm module (../src/amf/amf-sm.c:754)
03/26 20:31:54.913: [amf] INFO: [Added] Number of gNBs is now 2 (../src/amf/context.c:1236)
03/26 20:31:54.914: [amf] INFO: gNB-N2[192.168.0.132] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:793)
```

<a id="run_ue1"></a>

### Run UERANSIM (UE in Loc1)

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<a id="con_ue1"></a>

#### Start UE connected to gNodeB1 in Loc1

```
# ./nr-ue -c ../config/open5gs-ue-loc1.yaml 
UERANSIM v3.2.6
[2024-03-26 20:32:33.585] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-26 20:32:33.586] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-26 20:32:33.587] [nas] [info] Selected plmn[001/01]
[2024-03-26 20:32:33.588] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-26 20:32:33.589] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-26 20:32:33.589] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-26 20:32:33.590] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-26 20:32:33.593] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-26 20:32:33.593] [nas] [debug] Sending Initial Registration
[2024-03-26 20:32:33.594] [rrc] [debug] Sending RRC Setup Request
[2024-03-26 20:32:33.595] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-26 20:32:33.597] [rrc] [info] RRC connection established
[2024-03-26 20:32:33.597] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-26 20:32:33.598] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-26 20:32:33.625] [nas] [debug] Authentication Request received
[2024-03-26 20:32:33.626] [nas] [debug] Received SQN [000000000221]
[2024-03-26 20:32:33.626] [nas] [debug] SQN-MS [000000000000]
[2024-03-26 20:32:33.643] [nas] [debug] Security Mode Command received
[2024-03-26 20:32:33.644] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-26 20:32:33.687] [nas] [debug] Registration accept received
[2024-03-26 20:32:33.687] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-26 20:32:33.688] [nas] [debug] Sending Registration Complete
[2024-03-26 20:32:33.688] [nas] [info] Initial Registration is successful
[2024-03-26 20:32:33.688] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-26 20:32:33.689] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-26 20:32:33.892] [nas] [debug] Configuration Update Command received
[2024-03-26 20:32:33.954] [nas] [debug] PDU Session Establishment Accept received
[2024-03-26 20:32:33.960] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-26 20:32:34.011] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/26 20:32:33.577: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/26 20:32:33.577: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2656)
03/26 20:32:33.577: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/26 20:32:33.578: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1840)
03/26 20:32:33.578: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1621)
03/26 20:32:33.578: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/26 20:32:33.578: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/26 20:32:33.585: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.586: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.586: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.587: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.587: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.588: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.599: [sbi] INFO: [UDM] (SCP-discover) NF registered [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/path.c:211)
03/26 20:32:33.656: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.657: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.657: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.658: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.660: [sbi] INFO: [UDR] (SCP-discover) NF registered [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/path.c:211)
03/26 20:32:33.867: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/26 20:32:33.868: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/26 20:32:33.868: [gmm] INFO:     UTC [2024-03-26T11:32:33] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/26 20:32:33.869: [gmm] INFO:     LOCAL [2024-03-26T20:32:33] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/26 20:32:33.870: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/26 20:32:33.871: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/26 20:32:33.871: [gmm] INFO: SMF Instance [515c8168-eb64-41ee-b303-e141cb8d42b8] (../src/amf/gmm-handler.c:1324)
03/26 20:32:33.876: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/26 20:32:33.876: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/26 20:32:33.881: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.882: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.883: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.883: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.884: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.884: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.893: [sbi] INFO: [UDM] (SCP-discover) NF registered [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/path.c:211)
03/26 20:32:33.899: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.900: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.901: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.901: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.902: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.902: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.907: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.908: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.908: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.909: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.912: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [51485670-eb64-41ee-a5d7-4b79383d4067:2] (../lib/sbi/path.c:216)
03/26 20:32:33.915: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.916: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.916: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.917: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.920: [sbi] INFO: [BSF] (SCP-discover) NF registered [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/path.c:211)
03/26 20:32:33.923: [sbi] INFO: [PCF] (SCP-discover) NF registered [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/path.c:211)
03/26 20:32:33.923: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/26 20:32:33.926: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
03/26 20:32:33.937: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:32:33.938: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/26 20:32:33.938: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.939: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.939: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:32:33.939: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:32:33.943: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:2] (../lib/sbi/path.c:216)
03/26 20:32:33.944: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane1 log when executed is as follows.
```
03/26 20:32:33.918: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/26 20:32:33.918: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
03/26 20:32:33.918: [upf] INFO: UE F-SEID[UP:0x1ea CP:0x758] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/26 20:32:33.918: [upf] INFO: UE F-SEID[UP:0x1ea CP:0x758] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/26 20:32:33.928: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
6: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::c3c9:863b:ad40:5c2a/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue1"></a>

#### Ping google.com going through DN=10.45.0.0/16 on Loc1

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.175.78) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.175.78: icmp_seq=1 ttl=61 time=82.5 ms
64 bytes from 172.217.175.78: icmp_seq=2 ttl=61 time=94.6 ms
64 bytes from 172.217.175.78: icmp_seq=3 ttl=61 time=27.8 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
20:34:23.378745 IP 10.45.0.2 > 172.217.175.78: ICMP echo request, id 7, seq 1, length 64
20:34:23.457748 IP 172.217.175.78 > 10.45.0.2: ICMP echo reply, id 7, seq 1, length 64
20:34:24.379228 IP 10.45.0.2 > 172.217.175.78: ICMP echo request, id 7, seq 2, length 64
20:34:24.472158 IP 172.217.175.78 > 10.45.0.2: ICMP echo reply, id 7, seq 2, length 64
20:34:25.380573 IP 10.45.0.2 > 172.217.175.78: ICMP echo request, id 7, seq 3, length 64
20:34:25.405378 IP 172.217.175.78 > 10.45.0.2: ICMP echo reply, id 7, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 in the same Loc1 according to the connected gNodeB1 in Loc1.**

<a id="run_ue2"></a>

### Run UERANSIM (UE in Loc2)

Then the UE disconnects from gNodeB1 and connects to gNodeB2 in Loc2.
Confirm that the packet goes through the DN of U-Plane2 in the same Loc2.

<a id="con_ue2"></a>

#### Start UE connected to gNodeB2 in Loc2

```
# ./nr-ue -c ../config/open5gs-ue-loc2.yaml 
UERANSIM v3.2.6
[2024-03-26 20:35:16.314] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-26 20:35:16.316] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-26 20:35:16.317] [nas] [info] Selected plmn[001/01]
[2024-03-26 20:35:16.318] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2024-03-26 20:35:16.318] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-26 20:35:16.319] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-26 20:35:16.320] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-26 20:35:16.320] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-26 20:35:16.321] [nas] [debug] Sending Initial Registration
[2024-03-26 20:35:16.322] [rrc] [debug] Sending RRC Setup Request
[2024-03-26 20:35:16.323] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-26 20:35:16.324] [rrc] [info] RRC connection established
[2024-03-26 20:35:16.325] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-26 20:35:16.325] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-26 20:35:16.367] [nas] [debug] Authentication Request received
[2024-03-26 20:35:16.368] [nas] [debug] Received SQN [000000000241]
[2024-03-26 20:35:16.368] [nas] [debug] SQN-MS [000000000000]
[2024-03-26 20:35:16.384] [nas] [debug] Security Mode Command received
[2024-03-26 20:35:16.385] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-26 20:35:16.422] [nas] [debug] Registration accept received
[2024-03-26 20:35:16.422] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-26 20:35:16.422] [nas] [debug] Sending Registration Complete
[2024-03-26 20:35:16.423] [nas] [info] Initial Registration is successful
[2024-03-26 20:35:16.424] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-26 20:35:16.424] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-26 20:35:16.634] [nas] [debug] Configuration Update Command received
[2024-03-26 20:35:16.692] [nas] [debug] PDU Session Establishment Accept received
[2024-03-26 20:35:16.698] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-26 20:35:16.749] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/26 20:35:16.322: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/26 20:35:16.323: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2656)
03/26 20:35:16.323: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[2] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/26 20:35:16.323: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] known UE by SUCI (../src/amf/context.c:1838)
03/26 20:35:16.323: [amf] WARNING: [suci-0-001-01-0000-0-0-0000000000] Holding NG Context (../src/amf/amf-sm.c:965)
03/26 20:35:16.323: [amf] WARNING: [suci-0-001-01-0000-0-0-0000000000]    RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/amf-sm.c:965)
03/26 20:35:16.323: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/26 20:35:16.323: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/26 20:35:16.343: [smf] INFO: Removed Session: UE IMSI:[imsi-001010000000000] DNN:[internet:1] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1677)
03/26 20:35:16.344: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:3098)
03/26 20:35:16.345: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:1080)
03/26 20:35:16.346: [amf] INFO: [imsi-001010000000000:1] Release SM context [204] (../src/amf/amf-sm.c:505)
03/26 20:35:16.347: [amf] INFO: [imsi-001010000000000:1] Release SM Context [state:31] (../src/amf/nsmf-handler.c:1082)
03/26 20:35:16.347: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2684)
03/26 20:35:16.384: [gmm] WARNING: [suci-0-001-01-0000-0-0-0000000000] Clear NG Context (../src/amf/gmm-sm.c:2002)
03/26 20:35:16.385: [gmm] WARNING: [suci-0-001-01-0000-0-0-0000000000]    RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/gmm-sm.c:2002)
03/26 20:35:16.388: [amf] INFO: UE Context Release [Action:1] (../src/amf/ngap-handler.c:1696)
03/26 20:35:16.388: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/ngap-handler.c:1697)
03/26 20:35:16.389: [amf] INFO: [Removed] Number of gNB-UEs is now 1 (../src/amf/context.c:2663)
03/26 20:35:16.410: [pcf] WARNING: NF EndPoint(addr) updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:113)
03/26 20:35:16.627: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/26 20:35:16.627: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/26 20:35:16.628: [gmm] INFO:     UTC [2024-03-26T11:35:16] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/26 20:35:16.629: [gmm] INFO:     LOCAL [2024-03-26T20:35:16] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/26 20:35:16.630: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/26 20:35:16.630: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/26 20:35:16.631: [gmm] INFO: SMF Instance [515ca5ee-eb64-41ee-a438-79bc644a3a7f] (../src/amf/gmm-handler.c:1324)
03/26 20:35:16.634: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/26 20:35:16.635: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/26 20:35:16.638: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:35:16.639: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/26 20:35:16.639: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.640: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.640: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.641: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:35:16.649: [sbi] INFO: [UDM] (SCP-discover) NF registered [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/path.c:211)
03/26 20:35:16.654: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:35:16.655: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/26 20:35:16.656: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.657: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.657: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.658: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:35:16.662: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:35:16.663: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/26 20:35:16.664: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.664: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [51485670-eb64-41ee-a5d7-4b79383d4067:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:35:16.667: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [51485670-eb64-41ee-a5d7-4b79383d4067:2] (../lib/sbi/path.c:216)
03/26 20:35:16.670: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:35:16.671: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/26 20:35:16.671: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.672: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:35:16.674: [sbi] WARNING: [BSF] (SCP-discover) NF has already been added [513f1b32-eb64-41ee-9877-6137bf1512d7:1] (../lib/sbi/path.c:216)
03/26 20:35:16.677: [sbi] INFO: [PCF] (SCP-discover) NF registered [5148ef04-eb64-41ee-a175-3d43c24794a5:1] (../lib/sbi/path.c:211)
03/26 20:35:16.678: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/26 20:35:16.680: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
03/26 20:35:16.693: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1162)
03/26 20:35:16.694: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/26 20:35:16.694: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.694: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.695: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/26 20:35:16.695: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [5140b028-eb64-41ee-96db-57a830a03f36:1] (../lib/sbi/nnrf-handler.c:1200)
03/26 20:35:16.699: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [5140b028-eb64-41ee-96db-57a830a03f36:2] (../lib/sbi/path.c:216)
03/26 20:35:16.700: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane2 log when executed is as follows.
```
03/26 20:35:16.627: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/26 20:35:16.628: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
03/26 20:35:16.628: [upf] INFO: UE F-SEID[UP:0x8ae CP:0x874] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:485)
03/26 20:35:16.628: [upf] INFO: UE F-SEID[UP:0x8ae CP:0x874] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:485)
03/26 20:35:16.639: [gtp] INFO: gtp_connect() [192.168.0.132]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::251f:b652:cdee:a462/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue2"></a>

#### Ping google.com going through DN=10.46.0.0/16 on Loc2

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.222.14) from 10.46.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.222.14: icmp_seq=1 ttl=61 time=60.1 ms
64 bytes from 142.251.222.14: icmp_seq=2 ttl=61 time=49.9 ms
64 bytes from 142.251.222.14: icmp_seq=3 ttl=61 time=30.8 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
20:36:51.352123 IP 10.46.0.2 > 142.251.222.14: ICMP echo request, id 8, seq 1, length 64
20:36:51.409189 IP 142.251.222.14 > 10.46.0.2: ICMP echo reply, id 8, seq 1, length 64
20:36:52.353283 IP 10.46.0.2 > 142.251.222.14: ICMP echo request, id 8, seq 2, length 64
20:36:52.401124 IP 142.251.222.14 > 10.46.0.2: ICMP echo reply, id 8, seq 2, length 64
20:36:53.353019 IP 10.46.0.2 > 142.251.222.14: ICMP echo request, id 8, seq 3, length 64
20:36:53.382949 IP 142.251.222.14 > 10.46.0.2: ICMP echo reply, id 8, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.31] Removed `gtpc` section in `smf.yaml`.
- [2024.03.26] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.01.12] Updated to Open5GS v2.5.6.
- [2022.06.07] Updated to Open5GS v2.4.7 and UERANSIM v3.2.6.
- [2021.08.17] Initial release.
