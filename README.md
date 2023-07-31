# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select nearby UPF according to the connected gNodeB
This describes a very simple configuration that uses Open5GS and UERANSIM to select the nearby UPF according to the connected gNodeB.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
4. [Select nearby UPF(PGW-U) according to the connected eNodeB](https://github.com/s5uishida/open5gs_epc_srsran_nearby_upf_sample_config)
5. Select nearby UPF according to the connected gNodeB (this article)
6. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
7. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
8. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
9. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
10. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
11. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
12. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
---

<h2 id="misc">Miscellaneous Notes</h2>

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [A Note for Changing Network Interface of UPF from TUN to TAP in Open5GS](https://github.com/s5uishida/change_from_tun_to_tap_in_open5gs)
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
- 5GC - Open5GS v2.5.6 (2023.01.12) - https://github.com/open5gs/open5gs
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
- Open5GS v2.5.6 (2023.01.12) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2023-01-12 20:33:18.555295469 +0900
+++ amf.yaml    2023-01-12 22:23:40.014107855 +0900
@@ -342,26 +342,26 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.5
         port: 9090
     guami:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 999
-          mnc: 70
-        tac: 1
+          mcc: 001
+          mnc: 01
+        tac: [1, 2]
     plmn_support:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
     security:
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2023-01-12 20:33:18.526295426 +0900
+++ smf1.yaml   2023-01-12 22:24:13.992333976 +0900
@@ -19,7 +19,7 @@
 #    domain: core,fd,pfcp,gtp,smf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf1.log
 
 #
 # tls:
@@ -508,20 +508,17 @@
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
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -530,7 +527,17 @@
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
 # scp:
@@ -695,7 +702,8 @@
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
--- smf.yaml.orig       2023-01-12 20:33:18.526295426 +0900
+++ smf2.yaml   2023-01-12 22:24:24.289402198 +0900
@@ -19,7 +19,7 @@
 #    domain: core,fd,pfcp,gtp,smf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf2.log
 
 #
 # tls:
@@ -505,23 +505,20 @@
 #
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
     metrics:
-      - addr: 127.0.0.4
+      - addr: 127.0.0.24
         port: 9090
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -530,7 +527,17 @@
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
 # scp:
@@ -695,7 +702,8 @@
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
--- smf.conf.orig       2023-01-12 20:33:20.131297687 +0900
+++ smf2.conf   2023-01-12 22:22:40.352706816 +0900
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
--- upf.yaml.orig       2023-01-12 20:44:33.674609278 +0900
+++ upf.yaml    2023-01-12 22:26:40.722648205 +0900
@@ -173,12 +173,13 @@
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
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-01-12 20:53:25.948221315 +0900
+++ upf.yaml    2023-01-12 22:28:01.751564228 +0900
@@ -173,12 +173,13 @@
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
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran1">Changes in configuration files of RAN (gNodeB1)</h4>

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

<h4 id="changes_ran2">Changes in configuration files of RAN (gNodeB2)</h4>

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

<h4 id="changes_ue_loc1">Changes in configuration files of UE for Loc1 (IMSI-001010000000000)</h4>

- `UERANSIM/config/open5gs-ue-loc1.yaml`
```diff
--- open5gs-ue.yaml.orig        2022-07-03 13:06:43.000000000 +0900
+++ open5gs-ue-loc1.yaml        2023-01-12 22:33:47.598481682 +0900
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
--- open5gs-ue.yaml.orig        2022-07-03 13:06:43.000000000 +0900
+++ open5gs-ue-loc2.yaml        2023-01-12 22:33:56.743500724 +0900
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
- Open5GS v2.5.6 (2023.01.12) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

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
[2023-01-12 23:10:51.017] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2023-01-12 23:10:51.020] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2023-01-12 23:10:51.020] [sctp] [debug] SCTP association setup ascId[9]
[2023-01-12 23:10:51.020] [ngap] [debug] Sending NG Setup Request
[2023-01-12 23:10:51.021] [ngap] [debug] NG Setup Response received
[2023-01-12 23:10:51.021] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
01/12 23:10:51.006: [amf] INFO: gNB-N2 accepted[192.168.0.131]:39893 in ng-path module (../src/amf/ngap-sctp.c:113)
01/12 23:10:51.006: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:674)
01/12 23:10:51.007: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1034)
01/12 23:10:51.007: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:713)
```

<h4 id="run_ran2">Start gNodeB2 with TAC=2 in Loc2</h4>

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2023-01-12 23:11:54.993] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2023-01-12 23:11:54.996] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2023-01-12 23:11:54.996] [sctp] [debug] SCTP association setup ascId[4]
[2023-01-12 23:11:54.996] [ngap] [debug] Sending NG Setup Request
[2023-01-12 23:11:54.996] [ngap] [debug] NG Setup Response received
[2023-01-12 23:11:54.996] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
01/12 23:11:55.013: [amf] INFO: gNB-N2 accepted[192.168.0.132]:37118 in ng-path module (../src/amf/ngap-sctp.c:113)
01/12 23:11:55.013: [amf] INFO: gNB-N2 accepted[192.168.0.132] in master_sm module (../src/amf/amf-sm.c:674)
01/12 23:11:55.013: [amf] INFO: [Added] Number of gNBs is now 2 (../src/amf/context.c:1034)
01/12 23:11:55.013: [amf] INFO: gNB-N2[192.168.0.132] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:713)
```

<h3 id="run_ue1">Run UERANSIM (UE in Loc1)</h3>

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<h4 id="con_ue1">Start UE connected to gNodeB1 in Loc1</h4>

```
# ./nr-ue -c ../config/open5gs-ue-loc1.yaml 
UERANSIM v3.2.6
[2023-01-12 23:12:52.498] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-01-12 23:12:52.499] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-01-12 23:12:52.499] [nas] [info] Selected plmn[001/01]
[2023-01-12 23:12:52.500] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-01-12 23:12:52.500] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-01-12 23:12:52.500] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-01-12 23:12:52.500] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-01-12 23:12:52.502] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-01-12 23:12:52.502] [nas] [debug] Sending Initial Registration
[2023-01-12 23:12:52.502] [rrc] [debug] Sending RRC Setup Request
[2023-01-12 23:12:52.503] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-01-12 23:12:52.503] [rrc] [info] RRC connection established
[2023-01-12 23:12:52.503] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-01-12 23:12:52.503] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-01-12 23:12:52.512] [nas] [debug] Authentication Request received
[2023-01-12 23:12:52.517] [nas] [debug] Security Mode Command received
[2023-01-12 23:12:52.517] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-01-12 23:12:52.536] [nas] [debug] Registration accept received
[2023-01-12 23:12:52.537] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-01-12 23:12:52.537] [nas] [debug] Sending Registration Complete
[2023-01-12 23:12:52.537] [nas] [info] Initial Registration is successful
[2023-01-12 23:12:52.537] [nas] [debug] Sending PDU Session Establishment Request
[2023-01-12 23:12:52.537] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-01-12 23:12:52.742] [nas] [debug] Configuration Update Command received
[2023-01-12 23:12:52.765] [nas] [debug] PDU Session Establishment Accept received
[2023-01-12 23:12:52.768] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-01-12 23:12:52.791] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
01/12 23:12:52.493: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
01/12 23:12:52.493: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2322)
01/12 23:12:52.493: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:515)
01/12 23:12:52.493: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1629)
01/12 23:12:52.493: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1419)
01/12 23:12:52.493: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:546)
01/12 23:12:52.493: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
01/12 23:12:52.494: [sbi] WARNING: [cb0215a0-9282-41ed-ab11-6142b184f9dd] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.494: [sbi] WARNING: NF EndPoint updated [127.0.0.11:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.495: [sbi] WARNING: NF EndPoint updated [127.0.0.11:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.495: [sbi] INFO: [cb0215a0-9282-41ed-ab11-6142b184f9dd] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.496: [sbi] WARNING: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.496: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.496: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.497: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.497: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.497: [sbi] INFO: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.508: [sbi] WARNING: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.509: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.509: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.509: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.509: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.509: [sbi] INFO: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.512: [sbi] WARNING: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.512: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.512: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.512: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.512: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.513: [sbi] INFO: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.519: [sbi] WARNING: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.519: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.519: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.519: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.520: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.520: [sbi] INFO: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.522: [sbi] WARNING: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.522: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.523: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.523: [sbi] INFO: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.727: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1413)
01/12 23:12:52.728: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:435)
01/12 23:12:52.729: [gmm] INFO:     UTC [2023-01-12T14:12:52] Timezone[0]/DST[0] (../src/amf/gmm-build.c:543)
01/12 23:12:52.729: [gmm] INFO:     LOCAL [2023-01-12T23:12:52] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:548)
01/12 23:12:52.731: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2343)
01/12 23:12:52.731: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1084)
01/12 23:12:52.734: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1009)
01/12 23:12:52.734: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3088)
01/12 23:12:52.735: [sbi] WARNING: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.735: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.736: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.736: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.736: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.736: [sbi] INFO: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.741: [sbi] WARNING: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.741: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.741: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.742: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.742: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.742: [sbi] INFO: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.744: [sbi] WARNING: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.744: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.745: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.745: [sbi] INFO: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.746: [sbi] WARNING: [cb036ff4-9282-41ed-ab11-a5469b9ffc2e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:12:52.746: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1711)
01/12 23:12:52.747: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1623)
01/12 23:12:52.747: [sbi] INFO: [cb036ff4-9282-41ed-ab11-a5469b9ffc2e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:12:52.749: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:495)
01/12 23:12:52.750: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
```
The Open5GS U-Plane1 log when executed is as follows.
```
01/12 23:12:52.743: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:181)
01/12 23:12:52.743: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
01/12 23:12:52.743: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:401)
01/12 23:12:52.743: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:401)
01/12 23:12:52.748: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::1154:8c2f:e0f9:b5b8/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue1">Ping google.com going through DN=10.45.0.0/16 on Loc1</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.206) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.206: icmp_seq=1 ttl=61 time=20.6 ms
64 bytes from 142.251.42.206: icmp_seq=2 ttl=61 time=18.2 ms
64 bytes from 142.251.42.206: icmp_seq=3 ttl=61 time=18.7 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
23:15:04.363011 IP 10.45.0.2 > 142.251.42.206: ICMP echo request, id 3, seq 1, length 64
23:15:04.381573 IP 142.251.42.206 > 10.45.0.2: ICMP echo reply, id 3, seq 1, length 64
23:15:05.365527 IP 10.45.0.2 > 142.251.42.206: ICMP echo request, id 3, seq 2, length 64
23:15:05.381333 IP 142.251.42.206 > 10.45.0.2: ICMP echo reply, id 3, seq 2, length 64
23:15:06.366274 IP 10.45.0.2 > 142.251.42.206: ICMP echo request, id 3, seq 3, length 64
23:15:06.382585 IP 142.251.42.206 > 10.45.0.2: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 in the same Loc1 according to the connected gNodeB1 in Loc1.**

<h3 id="run_ue2">Run UERANSIM (UE in Loc2)</h3>

Then the UE disconnects from gNodeB1 and connects to gNodeB2 in Loc2.
Confirm that the packet goes through the DN of U-Plane2 in the same Loc2.

<h4 id="con_ue2">Start UE connected to gNodeB2 in Loc2</h4>

```
# ./nr-ue -c ../config/open5gs-ue-loc2.yaml 
UERANSIM v3.2.6
[2023-01-12 23:16:29.622] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-01-12 23:16:29.623] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-01-12 23:16:29.623] [nas] [info] Selected plmn[001/01]
[2023-01-12 23:16:29.624] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2023-01-12 23:16:29.624] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-01-12 23:16:29.624] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-01-12 23:16:29.624] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-01-12 23:16:29.626] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-01-12 23:16:29.626] [nas] [debug] Sending Initial Registration
[2023-01-12 23:16:29.626] [rrc] [debug] Sending RRC Setup Request
[2023-01-12 23:16:29.627] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-01-12 23:16:29.627] [rrc] [info] RRC connection established
[2023-01-12 23:16:29.627] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-01-12 23:16:29.627] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-01-12 23:16:29.637] [nas] [debug] Authentication Request received
[2023-01-12 23:16:29.642] [nas] [debug] Security Mode Command received
[2023-01-12 23:16:29.643] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-01-12 23:16:29.653] [nas] [debug] Registration accept received
[2023-01-12 23:16:29.653] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-01-12 23:16:29.653] [nas] [debug] Sending Registration Complete
[2023-01-12 23:16:29.653] [nas] [info] Initial Registration is successful
[2023-01-12 23:16:29.653] [nas] [debug] Sending PDU Session Establishment Request
[2023-01-12 23:16:29.653] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-01-12 23:16:29.862] [nas] [debug] Configuration Update Command received
[2023-01-12 23:16:29.885] [nas] [debug] PDU Session Establishment Accept received
[2023-01-12 23:16:29.891] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-01-12 23:16:29.912] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
01/12 23:16:29.608: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
01/12 23:16:29.609: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2322)
01/12 23:16:29.609: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[2] CellID[0x10] (../src/amf/ngap-handler.c:515)
01/12 23:16:29.609: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] known UE by SUCI (../src/amf/context.c:1627)
01/12 23:16:29.609: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:546)
01/12 23:16:29.609: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
01/12 23:16:29.609: [amf] INFO: UE Context Release [Action:1] (../src/amf/ngap-handler.c:1571)
01/12 23:16:29.609: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/ngap-handler.c:1572)
01/12 23:16:29.609: [amf] INFO: [Removed] Number of gNB-UEs is now 1 (../src/amf/context.c:2329)
01/12 23:16:29.612: [smf] INFO: Removed Session: UE IMSI:[imsi-001010000000000] DNN:[internet:1] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1707)
01/12 23:16:29.612: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:3096)
01/12 23:16:29.613: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:1068)
01/12 23:16:29.613: [amf] INFO: [imsi-001010000000000:1] Release SM context [204] (../src/amf/amf-sm.c:475)
01/12 23:16:29.613: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2350)
01/12 23:16:29.630: [pcf] WARNING: NF EndPoint updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:99)
01/12 23:16:29.839: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1413)
01/12 23:16:29.840: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:435)
01/12 23:16:29.840: [gmm] INFO:     UTC [2023-01-12T14:16:29] Timezone[0]/DST[0] (../src/amf/gmm-build.c:543)
01/12 23:16:29.841: [gmm] INFO:     LOCAL [2023-01-12T23:16:29] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:548)
01/12 23:16:29.842: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2343)
01/12 23:16:29.842: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1084)
01/12 23:16:29.845: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1009)
01/12 23:16:29.845: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3088)
01/12 23:16:29.847: [sbi] WARNING: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:16:29.848: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1711)
01/12 23:16:29.848: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.848: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.848: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.848: [sbi] INFO: [cb035410-9282-41ed-87a4-3f093df94d13] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:16:29.854: [sbi] WARNING: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:16:29.855: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1711)
01/12 23:16:29.855: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.855: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.855: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.855: [sbi] INFO: [cb098646-9282-41ed-a367-9973f4c7a0f6] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:16:29.856: [sbi] WARNING: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:16:29.857: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1711)
01/12 23:16:29.857: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.857: [sbi] INFO: [cb093452-9282-41ed-9ff5-71ae67f0fe04] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:16:29.859: [sbi] WARNING: [cb036ff4-9282-41ed-ab11-a5469b9ffc2e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:739)
01/12 23:16:29.859: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1711)
01/12 23:16:29.859: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1623)
01/12 23:16:29.859: [sbi] INFO: [cb036ff4-9282-41ed-ab11-a5469b9ffc2e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:762)
01/12 23:16:29.861: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:495)
01/12 23:16:29.862: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
```
The Open5GS U-Plane2 log when executed is as follows.
```
01/12 23:16:29.881: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:181)
01/12 23:16:29.881: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
01/12 23:16:29.881: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:401)
01/12 23:16:29.881: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:401)
01/12 23:16:29.886: [gtp] INFO: gtp_connect() [192.168.0.132]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
8: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::e033:44b5:6554:8f0c/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue2">Ping google.com going through DN=10.46.0.0/16 on Loc2</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.206) from 10.46.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.206: icmp_seq=1 ttl=61 time=20.6 ms
64 bytes from 142.251.42.206: icmp_seq=2 ttl=61 time=18.1 ms
64 bytes from 142.251.42.206: icmp_seq=3 ttl=61 time=18.6 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
23:18:26.481258 IP 10.46.0.2 > 142.251.42.206: ICMP echo request, id 4, seq 1, length 64
23:18:26.499338 IP 142.251.42.206 > 10.46.0.2: ICMP echo reply, id 4, seq 1, length 64
23:18:27.482355 IP 10.46.0.2 > 142.251.42.206: ICMP echo request, id 4, seq 2, length 64
23:18:27.498519 IP 142.251.42.206 > 10.46.0.2: ICMP echo reply, id 4, seq 2, length 64
23:18:28.484175 IP 10.46.0.2 > 142.251.42.206: ICMP echo request, id 4, seq 3, length 64
23:18:28.500714 IP 142.251.42.206 > 10.46.0.2: ICMP echo reply, id 4, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2023.01.12] Updated to Open5GS v2.5.6.
- [2022.06.07] Updated to Open5GS v2.4.7 and UERANSIM v3.2.6.
- [2021.08.17] Initial release.
