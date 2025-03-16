# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - ULCL(Uplink Classifier)
This describes a very simple configuration that uses free5GC and UERANSIM for ULCL(Uplink Classifier).  
Instead of [go-upf](https://github.com/free5gc/go-upf.git) (free5GC UPF), you may use [NextMN-UPF](https://github.com/nextmn/upf) as UPF to confirm ULCL.

**Note. For ULCL limitations of free5GC v4.0.0, please see [here](https://free5gc.org/guide/SMF-Config/).**

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of free5GC 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of free5GC 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of free5GC 5GC U-Plane (I-UPF)](#changes_up1)
  - [Changes in configuration files of free5GC 5GC U-Plane (PSA-UPF)](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB)](#changes_ran)
    - [Changes in configuration files of UE](#changes_ue)
- [Network settings of free5GC 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of free5GC 5GC U-Plane (I-UPF)](#network_settings_up1)
  - [Network settings of free5GC 5GC U-Plane (PSA-UPF)](#network_settings_up2)
- [Build free5GC and UERANSIM](#build)
- [Run free5GC 5GC and UERANSIM UE / RAN](#run)
  - [Run free5GC 5GC U-Plane (I-UPF & PSA-UPF)](#run_up)
  - [Run free5GC 5GC C-Plane](#run_cp)
  - [Run UERANSIM (gNodeB)](#run_ran)
  - [Run UERANSIM (UE)](#run_ue)
    - [Start UE connected to gNodeB](#con_ue)
    - [Ping google.com going through PSA-UPF](#ping_google)
    - [Ping 8.8.8.8 going through I-UPF](#ping_8)
    - [Ping 172.17.0.1 going through I-UPF](#ping_docker)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of free5GC 5GC Simulation Mobile Network

The following minimum configuration was set as a condition.
- I-UPF selects the communication paths according to the destination host and network.

The built simulation environment is as follows.
**In this configuration, I-UPF(VM2) must be reachable to Docker network on local.**

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v3.4.3 (2024.09.12) - https://github.com/free5gc/free5gc
- UPF - go-upf v1.2.3 (2024.05.11) - https://github.com/free5gc/go-upf
- (UPF) - gtp5g v0.8.10 (2024.06.03) - https://github.com/free5gc/gtp5g
- UE / RAN - UERANSIM v3.2.6 (2024.08.27) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Mem (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | free5GC  5GC C-Plane | 192.168.0.141/24 | Ubuntu 24.04 | 2GB | 20GB |
| VM2 | free5GC  5GC U-Plane (I-UPF) | 192.168.0.142/24 | Ubuntu 24.04 | 1GB | 10GB |
| VM3 | free5GC  5GC U-Plane (PSA-UPF) | 192.168.0.143/24 | Ubuntu 24.04 | 1GB | 10GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 24.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 24.04 | 1GB | 10GB |

Subscriber Information (other information is default) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE | IMSI | DNN | OP/OPc |
| --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc |

I registered these information with the free5GC WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

DN is as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- |
| 10.60.0.0/16 | upfgtp | internet | uesimtun0 |

The UE routing topology is as follows.
```
UE --- gNodeB --- I-UPF --- PSA-UPF
```

The communication paths to be confirmed for each destination IP address are as follows.
| Destination IP address | Communication path |
| --- | --- |
| google.com | I-UPF --> PSA-UPF --> Internet |
| 8.8.8.8 | I-UPF --> Internet |
| 172.17.0.1 | I-UPF --> 172.17.0.0/16 (Docker network on local) |

<a id="changes"></a>

## Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.4.3 (2024.09.12) - https://free5gc.org/guide/
- go-upf v1.2.3 (2024.05.11) - https://free5gc.org/guide/
- gtp5g v0.8.10 (2024.06.03) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.08.27) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

- `free5gc/config/amfcfg.yaml`
```diff
--- amfcfg.yaml.orig    2024-08-31 18:40:42.425497926 +0900
+++ amfcfg.yaml 2024-08-31 18:57:16.692270705 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.141
   ngapPort: 38412 # the SCTP port listened by NGAP
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
@@ -24,18 +24,18 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
   supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
   plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2024-08-31 18:40:42.425497926 +0900
+++ ausfcfg.yaml        2024-08-31 18:52:04.727832529 +0900
@@ -16,10 +16,8 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-    - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
   eapAkaSupiImsiPrefix: false # including "imsi-" prefix or not when using the SUPI to do EAP-AKA' authentication
 
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2024-08-31 18:40:42.425497926 +0900
+++ nrfcfg.yaml 2024-08-31 18:53:00.723935459 +0900
@@ -18,8 +18,8 @@
       key: cert/root.key
     oauth: true
   DefaultPlmnId:
-    mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-    mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+    mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   serviceNameList: # the SBI services provided by this NRF, refer to TS 29.510
     - nnrf-nfm # Nnrf_NFManagement service
     - nnrf-disc # Nnrf_NFDiscovery service
```
- `free5gc/config/nssfcfg.yaml`
```diff
--- nssfcfg.yaml.orig   2024-08-31 18:40:42.425497926 +0900
+++ nssfcfg.yaml        2024-08-31 18:58:49.940161382 +0900
@@ -18,12 +18,12 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   supportedPlmnList: # the PLMNs (Public land mobile network) list supported by this NSSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   supportedNssaiInPlmnList: # Supported S-NSSAI List for each PLMN
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       supportedSnssaiList: # Supported S-NSSAIs of the PLMN
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
```
- `free5gc/config/smfcfg.yaml`
```diff
--- smfcfg.yaml.orig    2024-09-15 18:09:13.892166385 +0900
+++ smfcfg.yaml 2024-09-15 18:13:33.913685244 +0900
@@ -34,14 +34,14 @@
             ipv4: 8.8.8.8
             ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.141 # the Node ID of this SMF
+    listenAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
     assocFailAlertInterval: 10s
     assocFailRetryInterval: 30s
     heartbeatInterval: 10s
@@ -49,38 +49,53 @@
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
-      UPF: # the name of the node
+      I-UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.142 # the Node ID of this UPF
+        addr: 192.168.0.142 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
               sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
-                pools:
-                  - cidr: 10.60.0.0/16
-                staticPools:
-                  - cidr: 10.60.100.0/24
+        interfaces: # Interface list for this UPF
+          - interfaceType: N3 # the type of the interface (N3 or N9)
+            endpoints: # the IP address of this N3/N9 interface on this UPF
+              - 192.168.0.142
+            networkInstances: # Data Network Name (DNN)
+              - internet
+          - interfaceType: N9 # the type of the interface (N3 or N9)
+            endpoints: # the IP address of this N3/N9 interface on this UPF
+              - 192.168.0.142
+            networkInstances: # Data Network Name (DNN)
+              - internet
+      PSA-UPF: # the name of the node
+        type: UPF # the type of the node (AN or UPF)
+        nodeID: 192.168.0.143 # the Node ID of this UPF
+        addr: 192.168.0.143 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
-                  - cidr: 10.61.0.0/16
+                  - cidr: 10.60.0.0/16
                 staticPools:
-                  - cidr: 10.61.100.0/24
+                  - cidr: 10.60.100.0/24
         interfaces: # Interface list for this UPF
-          - interfaceType: N3 # the type of the interface (N3 or N9)
+          - interfaceType: N9 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.143
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
       - A: gNB1
-        B: UPF
+        B: I-UPF
+      - A: I-UPF
+        B: PSA-UPF
+  ulcl: true
   # retransmission timer for pdu session modification command
   t3591:
     enable: true # true or false
```
- `free5gc/config/uerouting.yaml`
```yaml
info:
  version: 1.0.7
  description: Routing information for UE

ueRoutingInfo: # the list of UE routing information
  UE1: # Group Name
    members:
    - imsi-001010000000000 # Subscription Permanent Identifier of the UE
    topology: # Network topology for this group (Uplink: A->B, Downlink: B->A)
    # default path derived from this topology
    # node name should be consistent with smfcfg.yaml
      - A: gNB1
        B: I-UPF
      - A: I-UPF
        B: PSA-UPF
    specificPath:
      - dest: 8.8.8.8/32 # the destination IP address on Data Network (DN)
        # the order of UPF nodes in this path. We use the UPF's name to represent each UPF node.
        # The UPF's name should be consistent with smfcfg.yaml
        path: [I-UPF]
      - dest: 172.17.0.0/16
        path: [I-UPF]
```

<a id="changes_up1"></a>

### Changes in configuration files of free5GC 5GC U-Plane (I-UPF)

- `go-upf/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-08-31 19:23:52.419250932 +0900
+++ upfcfg.yaml 2024-09-15 18:28:12.128539749 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.142   # IP addr for listening
+  nodeID: 192.168.0.142 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,18 +13,18 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.142
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
       # mtu: 1400
+    - addr: 192.168.0.142
+      type: N9
 
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
     cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
-  - dnn: internet # Data Network Name
-    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_up2"></a>

### Changes in configuration files of free5GC 5GC U-Plane (PSA-UPF)

- `go-upf/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-08-31 19:23:52.419250932 +0900
+++ upfcfg.yaml 2024-09-15 18:32:10.552002281 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.143   # IP addr for listening
+  nodeID: 192.168.0.143 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,8 +13,8 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
-      type: N3
+    - addr: 192.168.0.143
+      type: N9
       # name: upf.5gc.nctu.me
       # ifname: gtpif
       # mtu: 1400
@@ -23,8 +23,6 @@
 dnnList:
   - dnn: internet # Data Network Name
     cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
-  - dnn: internet # Data Network Name
-    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN (gNodeB)

- `UERANSIM/config/free5gc-gnb.yaml`
```diff
--- free5gc-gnb.yaml.orig       2024-05-12 01:59:00.000000000 +0900
+++ free5gc-gnb.yaml    2024-09-15 18:42:02.193131540 +0900
@@ -1,17 +1,17 @@
-mcc: '208'          # Mobile Country Code value
-mnc: '93'           # Mobile Network Code value (2 or 3 digits)
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
-  - address: 127.0.0.1
+  - address: 192.168.0.141
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<a id="changes_ue"></a>

#### Changes in configuration files of UE

- `UERANSIM/config/free5gc-ue.yaml`
```diff
--- free5gc-ue.yaml.orig        2024-05-12 01:59:00.000000000 +0900
+++ free5gc-ue.yaml     2024-08-31 20:59:23.069355772 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
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

<a id="network_settings"></a>

## Network settings of free5GC 5GC and UERANSIM UE / RAN

<a id="network_settings_up1"></a>

### Network settings of free5GC 5GC U-Plane (I-UPF)

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="network_settings_up2"></a>

### Network settings of free5GC 5GC U-Plane (PSA-UPF)

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="build"></a>

## Build free5GC and UERANSIM

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.4.3 (2024.09.12) - https://free5gc.org/guide/
- go-upf v1.2.3 (2024.05.11) - https://github.com/s5uishida/install_goupf
- UERANSIM v3.2.6 (2024.08.27) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. The installation guide also includes instructions on building the latest committed version.
If it doesn't work properly and you suspect some bug, try updating all NFs by following this update procedure.**

<a id="run"></a>

## Run free5GC 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_up"></a>

### Run free5GC 5GC U-Plane (I-UPF & PSA-UPF)

- free5GC 5GC U-Plane (I-UPF)
```
# cd go-upf
# ./upf -c upfcfg.yaml
```
- free5GC 5GC U-Plane (PSA-UPF)
```
# cd go-upf
# ./upf -c upfcfg.yaml
```
Then run `tcpdump` on one more terminals for U-Plane.
- Run `tcpdump` on VM2 (U-Plane (I-UPF))
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), snapshot length 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane (PSA-UPF))
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), snapshot length 262144 bytes
```

<a id="run_cp"></a>

### Run free5GC 5GC C-Plane

Next, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```bash
#!/usr/bin/env bash

PID_LIST=()

NF_LIST="amf udr pcf udm nssf ausf chf"

export GIN_MODE=release

./bin/nrf &
PID_LIST+=($!)
sleep 1

./bin/smf -c config/smfcfg.yaml -u config/uerouting.yaml &
PID_LIST+=($!)
sleep 1

for NF in ${NF_LIST}; do
    ./bin/${NF} &
    PID_LIST+=($!)
    sleep 1
done

function terminate()
{
    sudo kill -SIGTERM ${PID_LIST[${#PID_LIST[@]}-2]} ${PID_LIST[${#PID_LIST[@]}-1]}
    sleep 2
}

trap terminate SIGINT
wait ${PID_LIST}
```

<a id="run_ran"></a>

### Run UERANSIM (gNodeB)

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2024-09-18 20:25:19.464] [sctp] [info] Trying to establish SCTP connection... (192.168.0.141:38412)
[2024-09-18 20:25:19.477] [sctp] [info] SCTP connection established (192.168.0.141:38412)
[2024-09-18 20:25:19.479] [sctp] [debug] SCTP association setup ascId[4]
[2024-09-18 20:25:19.480] [ngap] [debug] Sending NG Setup Request
[2024-09-18 20:25:19.495] [ngap] [debug] NG Setup Response received
[2024-09-18 20:25:19.496] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-09-18T20:25:19.484017097+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.131:46054
2024-09-18T20:25:19.487421209+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.131:46054
2024-09-18T20:25:19.493483395+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle NGSetupRequest
2024-09-18T20:25:19.494240597+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Send NG-Setup response
```

<a id="run_ue"></a>

### Run UERANSIM (UE)

Ping the following three destination IP addresses and confirm that they are routed through different UPFs.

- google.com - routing from PSA-UPF to Internet
- 8.8.8.8 - routing from I-UPF to Internet
- 172.17.0.1 - routing from I-UPF to 172.17.0.0/16 (Docker network on local)

**Note. For example, 172.17.0.0/16 is docker's default network.**

<a id="con_ue"></a>

#### Start UE connected to gNodeB

```
# ./nr-ue -c ../config/free5gc-ue.yaml
UERANSIM v3.2.6
[2024-09-18 20:26:07.587] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-09-18 20:26:07.589] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-09-18 20:26:07.591] [nas] [info] Selected plmn[001/01]
[2024-09-18 20:26:07.591] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-09-18 20:26:07.591] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-09-18 20:26:07.592] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-09-18 20:26:07.592] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-09-18 20:26:07.593] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-09-18 20:26:07.594] [nas] [debug] Sending Initial Registration
[2024-09-18 20:26:07.595] [rrc] [debug] Sending RRC Setup Request
[2024-09-18 20:26:07.595] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-09-18 20:26:07.597] [rrc] [info] RRC connection established
[2024-09-18 20:26:07.597] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-09-18 20:26:07.597] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-09-18 20:26:07.697] [nas] [debug] Authentication Request received
[2024-09-18 20:26:07.697] [nas] [debug] Received SQN [000000000042]
[2024-09-18 20:26:07.697] [nas] [debug] SQN-MS [000000000000]
[2024-09-18 20:26:07.726] [nas] [debug] Security Mode Command received
[2024-09-18 20:26:07.726] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-09-18 20:26:07.884] [nas] [debug] Registration accept received
[2024-09-18 20:26:07.884] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-09-18 20:26:07.884] [nas] [debug] Sending Registration Complete
[2024-09-18 20:26:07.884] [nas] [info] Initial Registration is successful
[2024-09-18 20:26:07.884] [nas] [debug] Sending PDU Session Establishment Request
[2024-09-18 20:26:07.885] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-09-18 20:26:08.093] [nas] [debug] Configuration Update Command received
[2024-09-18 20:26:08.298] [nas] [debug] PDU Session Establishment Accept received
[2024-09-18 20:26:08.298] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-09-18 20:26:08.319] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-09-18T20:26:07.632688035+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle InitialUEMessage
2024-09-18T20:26:07.632822271+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-09-18T20:26:07.632925869+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-09-18T20:26:07.636024889+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-09-18T20:26:07.637780236+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-09-18T20:26:07.638610707+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-09-18T20:26:07.638686787+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-09-18T20:26:07.639211373+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-09-18T20:26:07.640046263+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-09-18T20:26:07.640144354+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-09-18T20:26:07.643066359+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.649613275+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.660253767+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.665570430+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.669855280+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-09-18T20:26:07.672875975+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.676998230+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.686053063+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.689595567+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-09-18T20:26:07.690922488+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-09-18T20:26:07.692305174+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.693399113+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.697907865+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.699205166+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.700478804+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-09-18T20:26:07.701165015+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.702192448+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.705194883+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.706627221+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-09-18T20:26:07.707487385+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.708747193+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.711649944+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.712055752+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-09-18T20:26:07.712283815+09:00 [INFO][UDM][Suci] scheme 0
2024-09-18T20:26:07.712384480+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-09-18T20:26:07.713056795+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.715967655+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.718523474+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.720207873+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.721226276+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-09-18T20:26:07.723653123+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-09-18T20:26:07.724697132+09:00 [INFO][UDM][UEAU] Nil Op
2024-09-18T20:26:07.726834549+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-09-18T20:26:07.727028221+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-09-18T20:26:07.727288482+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-09-18T20:26:07.727320202+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-09-18T20:26:07.727334723+09:00 [INFO][AUSF][5gAka] XresStar = 3665656633323565623938363365653836613363623562383737616431343162
2024-09-18T20:26:07.727421796+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-09-18T20:26:07.728059414+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-09-18T20:26:07.728501625+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Send Downlink Nas Transport
2024-09-18T20:26:07.729165472+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-09-18T20:26:07.730499503+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport
2024-09-18T20:26:07.730838881+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-18T20:26:07.731163005+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-09-18T20:26:07.731473475+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-09-18T20:26:07.731492975+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-09-18T20:26:07.732242401+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.733850314+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.736800533+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.739445743+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-09-18T20:26:07.739489986+09:00 [INFO][AUSF][5gAka] res*: 3665656633323565623938363365653836613363623562383737616431343162
Xres*: 3665656633323565623938363365653836613363623562383737616431343162
2024-09-18T20:26:07.739508456+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-09-18T20:26:07.739960332+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.741745014+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.745290578+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.746808662+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-09-18T20:26:07.747788376+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.749203692+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.752332219+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.754540517+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-09-18T20:26:07.755134610+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-09-18T20:26:07.755520583+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-09-18T20:26:07.756298383+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-09-18T20:26:07.756407178+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-09-18T20:26:07.757227561+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Send Downlink Nas Transport
2024-09-18T20:26:07.757939334+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-09-18T20:26:07.759400747+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport
2024-09-18T20:26:07.759747020+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-18T20:26:07.760089322+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-09-18T20:26:07.760241963+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-09-18T20:26:07.760257305+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-09-18T20:26:07.760281094+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-09-18T20:26:07.760288303+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-09-18T20:26:07.761021270+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.762479001+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.765250663+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.766122422+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.767011349+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-09-18T20:26:07.767688441+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.768963745+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.772577525+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.773970597+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-09-18T20:26:07.775053173+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.776575367+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.779764263+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.781825852+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2024-09-18T20:26:07.784778525+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-09-18T20:26:07.786320190+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-18T20:26:07.787566579+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-09-18T20:26:07.787641212+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-09-18T20:26:07.788476126+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.790941882+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.793633004+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.794537905+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.795842313+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-09-18T20:26:07.797171961+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.798975509+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.802728209+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.804624352+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-09-18T20:26:07.805019730+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-09-18T20:26:07.805931520+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.806980341+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.809956417+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.812085590+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-09-18T20:26:07.812206803+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-09-18T20:26:07.813221243+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.814730153+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.817987677+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.819545000+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-09-18T20:26:07.820423909+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.821874250+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.825070580+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.826253382+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2024-09-18T20:26:07.827169135+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-18T20:26:07.827637730+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-18T20:26:07.828765031+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.829787994+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.833317342+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.834513245+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-09-18T20:26:07.835487152+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.836606356+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.839563009+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.840977827+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-09-18T20:26:07.841498829+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-18T20:26:07.842331417+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.843721384+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.846930179+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.848287196+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-09-18T20:26:07.849127994+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.850345065+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.853339112+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.854868609+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-09-18T20:26:07.855261635+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-09-18T20:26:07.856284225+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.857621203+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.860852596+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.862254516+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-09-18T20:26:07.863080024+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.864611595+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.868057100+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.871089460+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-09-18T20:26:07.871836455+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-09-18T20:26:07.872858305+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.874514863+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.877160918+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.877877174+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.878923667+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-09-18T20:26:07.879640031+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.880819521+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.884464676+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.886537867+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-09-18T20:26:07.887380381+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.888603287+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.891091433+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.891889345+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.892709379+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-09-18T20:26:07.893190355+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.894274490+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.897328469+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.898818580+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-09-18T20:26:07.899845269+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.900900267+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.903541545+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.904352144+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:07.905533741+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-09-18T20:26:07.906890620+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:07.907970989+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:07.911091323+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:07.912491589+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-09-18T20:26:07.912540218+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2024-09-18T20:26:07.912584859+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-09-18T20:26:07.913123219+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-09-18T20:26:07.913685717+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-09-18T20:26:07.913999336+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Send Initial Context Setup Request
2024-09-18T20:26:07.915418342+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-09-18T20:26:07.916286704+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle InitialContextSetupResponse
2024-09-18T20:26:07.916461669+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-09-18T20:26:08.122470807+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport
2024-09-18T20:26:08.123036234+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-18T20:26:08.123112168+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-09-18T20:26:08.123124641+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-09-18T20:26:08.123134748+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-09-18T20:26:08.123166967+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-09-18T20:26:08.123179384+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Send Downlink Nas Transport
2024-09-18T20:26:08.124427508+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-09-18T20:26:08.127248938+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport
2024-09-18T20:26:08.128199436+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-18T20:26:08.128930708+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-09-18T20:26:08.129795661+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-09-18T20:26:08.129857216+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-09-18T20:26:08.129912778+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-09-18T20:26:08.132971410+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.139676751+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.150864277+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.153846614+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.157864192+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-09-18T20:26:08.159964624+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.166161214+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.176383752+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.181645836+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-09-18T20:26:08.182684170+09:00 [WARN][NSSF][Util] No TA {"plmnId":{"mcc":"001","mnc":"01"},"tac":"000001"} in NSSF configuration
2024-09-18T20:26:08.183371597+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=96b608e3-15d4-47db-a283-39860b765884&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D&tai=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22tac%22%3A%22000001%22%7D |
2024-09-18T20:26:08.184544370+09:00 [WARN][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] nsiInformation is still nil, use default NRF[http://127.0.0.10:8000]
2024-09-18T20:26:08.186222787+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.189118322+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.193706776+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.195255090+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.197462457+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-18T20:26:08.198530983+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.200635575+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.205771506+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.208180968+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-09-18T20:26:08.209368924+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-09-18T20:26:08.209820343+09:00 [INFO][SMF][CTX] UrrPeriod: 30s
2024-09-18T20:26:08.210077449+09:00 [INFO][SMF][CTX] UrrThreshold: 500000
2024-09-18T20:26:08.211005437+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.212777439+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.216063987+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.217075041+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.218396565+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-09-18T20:26:08.219041521+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-09-18T20:26:08.219689423+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.220954312+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.224647776+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.226315668+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-09-18T20:26:08.227199614+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.228721197+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.232080821+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.232557943+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-09-18T20:26:08.234811314+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-18T20:26:08.235500018+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-18T20:26:08.236390805+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-09-18T20:26:08+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0002dcf20 0xc0002dcf40]
2024-09-18T20:26:08.236931651+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-09-18T20:26:08.237372537+09:00 [INFO][SMF][GSM] &{[0xc0002dcf20 0xc0002dcf40]}
2024-09-18T20:26:08.237578793+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-09-18T20:26:08.238551521+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.239626502+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.242259993+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.243129888+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.244421037+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=96b608e3-15d4-47db-a283-39860b765884&target-nf-type=AMF |
2024-09-18T20:26:08.244910170+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-09-18T20:26:08.245410741+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-09-18T20:26:08.245627427+09:00 [INFO][SMF][CTX] Selected UPF: PSA-UPF
2024-09-18T20:26:08.246099667+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.247130868+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.249661674+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.250781047+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.251642158+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2024-09-18T20:26:08.252672018+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.254763133+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.259168070+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.260393656+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-09-18T20:26:08.261244879+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.262811400+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.265816595+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.268036899+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-18T20:26:08.272737950+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.274095813+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.277125219+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.278786056+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-09-18T20:26:08.279095086+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-09-18T20:26:08.279942914+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.281358707+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.284397640+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.285894136+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-09-18T20:26:08.286762573+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.289522625+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.292067981+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.294361736+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.295105422+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-09-18T20:26:08.296000968+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-09-18T20:26:08.297116374+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-09-18T20:26:08.297951527+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.299169189+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.301610858+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.302407473+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-18T20:26:08.303209308+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-09-18T20:26:08.303522083+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-09-18T20:26:08.303800028+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.304706321+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.307649125+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.311548011+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-09-18T20:26:08.311812081+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-09-18T20:26:08.312161840+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2121: connect: connection refused
2024-09-18T20:26:08.312470483+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-09-18T20:26:08.312750860+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-09-18T20:26:08.313164785+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-09-18T20:26:08.313889068+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-09-18T20:26:08.314342582+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-09-18T20:26:08.314566963+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-09-18T20:26:08.314790087+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-09-18T20:26:08.314811473+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-09-18T20:26:08.314859720+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has pre-config route
2024-09-18T20:26:08.315232455+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-09-18T20:26:08.315704620+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-09-18T20:26:08.315910457+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-09-18T20:26:08.316273764+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-09-18T20:26:08.318191885+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-09-18T20:26:08.319106567+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-09-18T20:26:08.320856593+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.322058557+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.325421741+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.327230844+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-09-18T20:26:08.327746280+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Send PDU Session Resource Setup Request
2024-09-18T20:26:08.328816178+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-09-18T20:26:08.330854297+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:46054] Handle PDUSessionResourceSetupResponse
2024-09-18T20:26:08.331151783+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:46054] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-09-18T20:26:08.331961778+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-18T20:26:08.333484755+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-18T20:26:08.336701596+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-18T20:26:08.338103118+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-09-18T20:26:08.340326055+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-09-18T20:26:08.340587188+09:00 [INFO][SMF][PFCP] Add PSAAndULCL
2024-09-18T20:26:08.340600318+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] In AddPDUSessionAnchorAndULCL
2024-09-18T20:26:08.340629015+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Establish PSA2
2024-09-18T20:26:08.340638067+09:00 [INFO][SMF][PduSess] In EstablishULCL
2024-09-18T20:26:08.340667495+09:00 [INFO][SMF][PFCP] [SMF] Establish ULCL msg has been send
2024-09-18T20:26:08.340674189+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request
2024-09-18T20:26:08.344325528+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Response
2024-09-18T20:26:08.344370934+09:00 [INFO][SMF][CTX] [SMF] Add PSA success
2024-09-18T20:26:08.344427257+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Establish PSA2
2024-09-18T20:26:08.344580049+09:00 [INFO][SMF][PduSess] In EstablishULCL
2024-09-18T20:26:08.344636187+09:00 [INFO][SMF][PFCP] [SMF] Establish ULCL msg has been send
2024-09-18T20:26:08.344682288+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request
2024-09-18T20:26:08.346959847+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Response
2024-09-18T20:26:08.347220696+09:00 [INFO][SMF][CTX] [SMF] Add PSA success
2024-09-18T20:26:08.347447790+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:b899493b-c294-4c81-ae90-2b35c8bc8e88/modify |
```
The free5GC U-Plane (I-UPF) log when executed is as follows.
```
2024-09-18T20:26:08.324560922+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionEstablishmentRequest
2024-09-18T20:26:08.324590034+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] New session
2024-09-18T20:26:08.325641181+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:5 period:30000000000}]
2024-09-18T20:26:08.325830591+09:00 [INFO][UPF][Perio] new ticker [30s]
2024-09-18T20:26:08.326080274+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:6 period:30000000000}]
2024-09-18T20:26:08.347054789+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-18T20:26:08.348803361+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-18T20:26:08.350750974+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:30000000000}]
2024-09-18T20:26:08.351176910+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:30000000000}]
2024-09-18T20:26:08.352908387+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-18T20:26:08.353610919+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:30000000000}]
2024-09-18T20:26:08.353803414+09:00 [ERRO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] Mod CreateURR error: file exists
2024-09-18T20:26:08.353880210+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:30000000000}]
2024-09-18T20:26:08.353924199+09:00 [ERRO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] Mod CreateURR error: file exists
```
The free5GC U-Plane (PSA-UPF) log when executed is as follows.
```
2024-09-18T20:26:08.282795607+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.143:8805] handleSessionEstablishmentRequest
2024-09-18T20:26:08.282831771+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.143:8805][CPNodeID:192.168.0.141][CPSEID:0x2][UPSEID:0x1] New session
2024-09-18T20:26:08.283807070+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:3 period:30000000000}]
2024-09-18T20:26:08.284000580+09:00 [INFO][UPF][Perio] new ticker [30s]
2024-09-18T20:26:08.284079651+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:4 period:30000000000}]
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
5: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::81f5:e632:81e2:64af/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_google"></a>

#### Ping google.com going through PSA-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (PSA-UPF).
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.222.46) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.222.46: icmp_seq=1 ttl=61 time=21.5 ms
64 bytes from 142.251.222.46: icmp_seq=2 ttl=61 time=22.6 ms
64 bytes from 142.251.222.46: icmp_seq=3 ttl=61 time=20.0 ms
```
The `tcpdump` log on U-Plane (PSA-UPF) is as follows.
```
20:34:46.778014 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1410, seq 1, length 64
20:34:46.797300 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1410, seq 1, length 64
20:34:47.781585 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1410, seq 2, length 64
20:34:47.800842 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1410, seq 2, length 64
20:34:48.782556 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1410, seq 3, length 64
20:34:48.799481 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1410, seq 3, length 64
```
**Note. Make sure that the packets are not routed from I-UPF to the Internet.**

<a id="ping_8"></a>

#### Ping 8.8.8.8 going through I-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (I-UPF).
```
# ping 8.8.8.8 -I uesimtun0 -n
PING 8.8.8.8 (8.8.8.8) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=11.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=61 time=11.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=61 time=12.5 ms
```
The `tcpdump` log on U-Plane (I-UPF) is as follows.
```
20:36:04.026119 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1415, seq 1, length 64
20:36:04.035591 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1415, seq 1, length 64
20:36:05.028148 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1415, seq 2, length 64
20:36:05.037062 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1415, seq 2, length 64
20:36:06.028884 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1415, seq 3, length 64
20:36:06.038058 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1415, seq 3, length 64
```
**Note. Make sure that the packets are not routed from PSA-UPF to the Internet.**

<a id="ping_docker"></a>

#### Ping 172.17.0.1 going through I-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (I-UPF).
```
# ping 172.17.0.1 -I uesimtun0 -n
PING 172.17.0.1 (172.17.0.1) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=2.62 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=2.46 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=2.52 ms
```
The `tcpdump` log on U-Plane (I-UPF) is as follows.
```
20:37:07.525065 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1416, seq 1, length 64
20:37:07.525200 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1416, seq 1, length 64
20:37:08.526719 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1416, seq 2, length 64
20:37:08.526821 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1416, seq 2, length 64
20:37:09.527729 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1416, seq 3, length 64
20:37:09.527832 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1416, seq 3, length 64
```
**Note. Make sure that the packets are not routed from PSA-UPF to anywhere.**

---
Using ULCL, I was able to confirm the very simple configuration for controlling the communication path to specific destination IP addresses.
I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2025.03.16] Added the information about ULCL limitations in free5GC v4.0.0.
- [2024.09.18] According to [this](https://github.com/free5gc/free5gc/issues/599#issuecomment-2357448799), the bug in the ULCL function of SMF has been fixed, so updated to go-upf v1.2.3.
- [2024.09.16] Added a note at the beginning that ULCL can be confirmed using [NextMN-UPF](https://github.com/nextmn/upf) as UPF.
- [2024.09.15] Updated to free5GC v3.4.3 (2024.09.12) and go-upf v1.2.1 (2023.12.19).
- [2022.08.16] Initial release.
