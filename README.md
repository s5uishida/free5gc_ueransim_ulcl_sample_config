# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - ULCL(Uplink Classifier)
This describes a very simple configuration that uses free5GC and UERANSIM for ULCL(Uplink Classifier).  
**Note. According to [this](https://forum.free5gc.org/t/access-to-local-dn-through-i-upf-in-ulcl-scenario/2512/4), the ULCL feature in free5GC v3.4.0 or later versions won’t work due to the lack of forwarding parameters in FAR creation.**

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
  - [Build go-upf](#build_upf)
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
- UPF - go-upf v1.2.1 (2023.12.19) - https://github.com/free5gc/go-upf
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
- **UE** -- **gNodeB** -- **I-UPF** -- **PSA-UPF**

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
- go-upf v1.2.1 (2023.12.19) - https://free5gc.org/guide/
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

**Note. It is recommended to use go1.18.x according to the commit to free5gc/openapi on 2022.10.26.**

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.4.3 (2024.09.12) - https://free5gc.org/guide/
- go-upf v1.2.1 (2023.12.19) - https://free5gc.org/guide/
- gtp5g v0.8.10 (2024.06.03) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.08.27) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. The installation guide also includes instructions on building the latest committed version.**

<a id="build_upf"></a>

### Build go-upf

For UPF, select the go-upf version that enables ULCL with the free5GC version selected this time.
Please refer to the free5GC guide at the above URL for building free5GC UPF. Below show only the differences in the procedure.
```
# git clone -b v1.2.1 https://github.com/free5gc/go-upf.git
# cd go-upf
# CGO_ENABLED=0 go build -o upf cmd/main.go
# ls upf
upf
# wget https://raw.githubusercontent.com/free5gc/free5gc/main/config/upfcfg.yaml
```

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
[2024-09-15 20:16:45.677] [sctp] [info] Trying to establish SCTP connection... (192.168.0.141:38412)
[2024-09-15 20:16:45.690] [sctp] [info] SCTP connection established (192.168.0.141:38412)
[2024-09-15 20:16:45.691] [sctp] [debug] SCTP association setup ascId[4]
[2024-09-15 20:16:45.692] [ngap] [debug] Sending NG Setup Request
[2024-09-15 20:16:45.705] [ngap] [debug] NG Setup Response received
[2024-09-15 20:16:45.705] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-09-15T20:16:45.678092559+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.131:40751
2024-09-15T20:16:45.681412529+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.131:40751
2024-09-15T20:16:45.686132914+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle NGSetupRequest
2024-09-15T20:16:45.686692209+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Send NG-Setup response
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
[2024-09-15 20:17:45.161] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-09-15 20:17:45.163] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-09-15 20:17:45.164] [nas] [info] Selected plmn[001/01]
[2024-09-15 20:17:45.165] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-09-15 20:17:45.166] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-09-15 20:17:45.166] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-09-15 20:17:45.166] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-09-15 20:17:45.167] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-09-15 20:17:45.167] [nas] [debug] Sending Initial Registration
[2024-09-15 20:17:45.167] [rrc] [debug] Sending RRC Setup Request
[2024-09-15 20:17:45.168] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-09-15 20:17:45.170] [rrc] [info] RRC connection established
[2024-09-15 20:17:45.170] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-09-15 20:17:45.170] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-09-15 20:17:45.289] [nas] [debug] Authentication Request received
[2024-09-15 20:17:45.289] [nas] [debug] Received SQN [00000000003A]
[2024-09-15 20:17:45.289] [nas] [debug] SQN-MS [000000000000]
[2024-09-15 20:17:45.318] [nas] [debug] Security Mode Command received
[2024-09-15 20:17:45.318] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-09-15 20:17:45.471] [nas] [debug] Registration accept received
[2024-09-15 20:17:45.471] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-09-15 20:17:45.471] [nas] [debug] Sending Registration Complete
[2024-09-15 20:17:45.471] [nas] [info] Initial Registration is successful
[2024-09-15 20:17:45.471] [nas] [debug] Sending PDU Session Establishment Request
[2024-09-15 20:17:45.471] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-09-15 20:17:45.681] [nas] [debug] Configuration Update Command received
[2024-09-15 20:17:45.868] [nas] [debug] PDU Session Establishment Accept received
[2024-09-15 20:17:45.868] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-09-15 20:17:45.891] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-09-15T20:17:45.170868990+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle InitialUEMessage
2024-09-15T20:17:45.171451544+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-09-15T20:17:45.171905327+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-09-15T20:17:45.175443379+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-09-15T20:17:45.176634358+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-09-15T20:17:45.177534027+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-09-15T20:17:45.177796423+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-09-15T20:17:45.178848979+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-09-15T20:17:45.178924521+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-09-15T20:17:45.178970520+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-09-15T20:17:45.182804265+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.189249656+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.200025455+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.204083524+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.207417696+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-09-15T20:17:45.211011101+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.215179232+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.223973770+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.228176910+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-09-15T20:17:45.228287657+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-09-15T20:17:45.229885185+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.232231135+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.237894854+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.239651761+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.241983467+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-09-15T20:17:45.243552581+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.245202186+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.251571106+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.253855257+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-09-15T20:17:45.255409864+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.257575761+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.262439826+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.265462705+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-09-15T20:17:45.265737300+09:00 [INFO][UDM][Suci] scheme 0
2024-09-15T20:17:45.265759505+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-09-15T20:17:45.266673359+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.268767248+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.272096485+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.273551329+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.274689830+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-09-15T20:17:45.277722823+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-09-15T20:17:45.278843232+09:00 [INFO][UDM][UEAU] Nil Op
2024-09-15T20:17:45.281544082+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-09-15T20:17:45.282208457+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-09-15T20:17:45.282566429+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-09-15T20:17:45.283033338+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-09-15T20:17:45.283242418+09:00 [INFO][AUSF][5gAka] XresStar = 3862343536383931363461303064643633633439323237363364643335663435
2024-09-15T20:17:45.283365221+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-09-15T20:17:45.284281240+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-09-15T20:17:45.284374906+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Send Downlink Nas Transport
2024-09-15T20:17:45.285170965+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-09-15T20:17:45.286672679+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport
2024-09-15T20:17:45.286967436+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.287359745+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-09-15T20:17:45.287656670+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-09-15T20:17:45.287943800+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-09-15T20:17:45.288840555+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.290528898+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.293854168+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.295408329+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-09-15T20:17:45.295871411+09:00 [INFO][AUSF][5gAka] res*: 3862343536383931363461303064643633633439323237363364643335663435
Xres*: 3862343536383931363461303064643633633439323237363364643335663435
2024-09-15T20:17:45.296423324+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-09-15T20:17:45.297104247+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.298012825+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.301669706+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.303158077+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-09-15T20:17:45.304131050+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.305430739+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.308713584+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.311855112+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-09-15T20:17:45.312159183+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-09-15T20:17:45.312831382+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-09-15T20:17:45.313507187+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-09-15T20:17:45.313726904+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-09-15T20:17:45.313806488+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Send Downlink Nas Transport
2024-09-15T20:17:45.314530996+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-09-15T20:17:45.316636785+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport
2024-09-15T20:17:45.317150437+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.317476701+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-09-15T20:17:45.317844869+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-09-15T20:17:45.318145752+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-09-15T20:17:45.318692307+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-09-15T20:17:45.318865110+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-09-15T20:17:45.320209326+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.321857463+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.324717788+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.325624912+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.326693768+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-09-15T20:17:45.327428224+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.328753137+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.332154826+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.333581550+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-09-15T20:17:45.334538700+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.335886188+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.339271310+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.340625060+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2024-09-15T20:17:45.341276859+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-09-15T20:17:45.341750055+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-15T20:17:45.342177832+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-09-15T20:17:45.342232236+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-09-15T20:17:45.342912976+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.345118313+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.347738062+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.350485689+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.351668722+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-09-15T20:17:45.352450013+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.353848009+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.357115379+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.358345522+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-09-15T20:17:45.358667457+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-09-15T20:17:45.359521603+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.360662063+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.363647756+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.369060990+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-09-15T20:17:45.369193794+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-09-15T20:17:45.370214505+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.371730089+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.374957453+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.375994598+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-09-15T20:17:45.376852096+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.377924029+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.380848387+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.381837139+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2024-09-15T20:17:45.382529840+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-15T20:17:45.382944495+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-15T20:17:45.384828736+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.387274937+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.391157036+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.392456297+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-09-15T20:17:45.393244409+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.394411386+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.397399378+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.398865756+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-09-15T20:17:45.399235344+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-15T20:17:45.400387523+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.401701551+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.405397509+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.408320339+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-09-15T20:17:45.409263836+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.410491466+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.413466929+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.414989996+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-09-15T20:17:45.415517377+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-09-15T20:17:45.416587293+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.417824432+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.421103907+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.422320907+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-09-15T20:17:45.422888596+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.424009391+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.426961380+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.428225405+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-09-15T20:17:45.428940128+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-09-15T20:17:45.429848976+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.431151326+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.433914106+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.434750939+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.435900122+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-09-15T20:17:45.436568844+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.437646315+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.440680443+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.442616991+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-09-15T20:17:45.443481360+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.444571359+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.447101673+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.447991351+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.448896537+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-09-15T20:17:45.449542306+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.450701724+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.453707592+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.455498109+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-09-15T20:17:45.456627564+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.458701473+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.461250918+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.463473932+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.464701346+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-09-15T20:17:45.465515890+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-09-15T20:17:45.466158536+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-09-15T20:17:45.466213521+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Send Initial Context Setup Request
2024-09-15T20:17:45.467592138+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-09-15T20:17:45.468392030+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle InitialContextSetupResponse
2024-09-15T20:17:45.468679798+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.673526450+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport
2024-09-15T20:17:45.673822852+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.673978052+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-09-15T20:17:45.674008840+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-09-15T20:17:45.674031860+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-09-15T20:17:45.674092517+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-09-15T20:17:45.674117289+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Send Downlink Nas Transport
2024-09-15T20:17:45.676927320+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-09-15T20:17:45.681107363+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport
2024-09-15T20:17:45.681259535+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.681563826+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-09-15T20:17:45.682799323+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-09-15T20:17:45.682912614+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-09-15T20:17:45.683072327+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-09-15T20:17:45.686021084+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.691972851+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.703060695+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.705459705+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.708919458+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-09-15T20:17:45.710647050+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.714729354+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.723302671+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.727066706+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-09-15T20:17:45.727239014+09:00 [WARN][NSSF][Util] No TA {"plmnId":{"mcc":"001","mnc":"01"},"tac":"000001"} in NSSF configuration
2024-09-15T20:17:45.728379313+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=76cd45a4-ebbe-465a-a5ed-567658ad960c&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D&tai=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22tac%22%3A%22000001%22%7D |
2024-09-15T20:17:45.730074401+09:00 [WARN][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] nsiInformation is still nil, use default NRF[http://127.0.0.10:8000]
2024-09-15T20:17:45.731656635+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.734520091+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.739188738+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.740242668+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.742489928+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-09-15T20:17:45.744212311+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.745139856+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.749223106+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.750799969+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-09-15T20:17:45.751650292+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-09-15T20:17:45.751953807+09:00 [INFO][SMF][CTX] UrrPeriod: 30s
2024-09-15T20:17:45.752328348+09:00 [INFO][SMF][CTX] UrrThreshold: 500000
2024-09-15T20:17:45.753106835+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.754262198+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.756746408+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.757603384+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.758739489+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-09-15T20:17:45.759344094+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-09-15T20:17:45.760812767+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.762691355+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.765924445+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.767075704+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-09-15T20:17:45.767855253+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.768988057+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.771981674+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.772514414+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-09-15T20:17:45.774157632+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-15T20:17:45.774859677+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-15T20:17:45.775406716+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-09-15T20:17:45+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0002de080 0xc0002de0a0]
2024-09-15T20:17:45.775896510+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-09-15T20:17:45.775945449+09:00 [INFO][SMF][GSM] &{[0xc0002de080 0xc0002de0a0]}
2024-09-15T20:17:45.775977628+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-09-15T20:17:45.776546574+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.777628500+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.780044179+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.780945323+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.782322536+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=76cd45a4-ebbe-465a-a5ed-567658ad960c&target-nf-type=AMF |
2024-09-15T20:17:45.783014137+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-09-15T20:17:45.783178743+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-09-15T20:17:45.783218673+09:00 [INFO][SMF][CTX] Selected UPF: PSA-UPF
2024-09-15T20:17:45.783521051+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.784658928+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.787175803+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.787928493+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.789081572+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2024-09-15T20:17:45.789809565+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.790884483+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.794107637+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.795514662+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-09-15T20:17:45.796547763+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.798017498+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.800919346+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.803580755+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-09-15T20:17:45.808072270+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.811070375+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.815112482+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.816864995+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-09-15T20:17:45.817291638+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-09-15T20:17:45.818094962+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.819424054+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.822164593+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.823382712+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-09-15T20:17:45.824208556+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.825533833+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.828042354+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.828861936+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.829502337+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-09-15T20:17:45.830502453+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-09-15T20:17:45.831607754+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-09-15T20:17:45.832090774+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.833172464+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.835690297+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.836489863+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-09-15T20:17:45.837327636+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-09-15T20:17:45.837758221+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-09-15T20:17:45.838173068+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.839318331+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.842166677+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.846118412+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-09-15T20:17:45.846420906+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-09-15T20:17:45.846774302+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2121: connect: connection refused
2024-09-15T20:17:45.846829529+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-09-15T20:17:45.846963549+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-09-15T20:17:45.847189853+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-09-15T20:17:45.847863970+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-09-15T20:17:45.848178832+09:00 [INFO][SMF][CTX] No Default Data Path
2024-09-15T20:17:45.848508143+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-09-15T20:17:45.848548601+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-09-15T20:17:45.848585588+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-09-15T20:17:45.848600623+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-09-15T20:17:45.848819170+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has pre-config route
2024-09-15T20:17:45.849365889+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-09-15T20:17:45.849691742+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-09-15T20:17:45.849696044+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-09-15T20:17:45.850977730+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-09-15T20:17:45.853143779+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-09-15T20:17:45.853416517+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-09-15T20:17:45.855267273+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.856644179+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.860324807+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.862300334+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-09-15T20:17:45.862474459+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Send PDU Session Resource Setup Request
2024-09-15T20:17:45.863509872+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-09-15T20:17:45.865628262+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:40751] Handle PDUSessionResourceSetupResponse
2024-09-15T20:17:45.865937548+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:40751] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-09-15T20:17:45.866933428+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-09-15T20:17:45.868292586+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-09-15T20:17:45.871678318+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-09-15T20:17:45.873283309+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-09-15T20:17:45.875688984+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-09-15T20:17:45.875746988+09:00 [INFO][SMF][PFCP] Add PSAAndULCL
2024-09-15T20:17:45.875759746+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] In AddPDUSessionAnchorAndULCL
2024-09-15T20:17:45.875969259+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Establish PSA2
2024-09-15T20:17:45.876192940+09:00 [INFO][SMF][PduSess] In EstablishULCL
2024-09-15T20:17:45.876334715+09:00 [INFO][SMF][PFCP] [SMF] Establish ULCL msg has been send
2024-09-15T20:17:45.876653667+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request
2024-09-15T20:17:45.878786491+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Response
2024-09-15T20:17:45.878941385+09:00 [INFO][SMF][CTX] [SMF] Add PSA success
2024-09-15T20:17:45.878991323+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Establish PSA2
2024-09-15T20:17:45.879141516+09:00 [INFO][SMF][PduSess] In EstablishULCL
2024-09-15T20:17:45.879332704+09:00 [INFO][SMF][PFCP] [SMF] Establish ULCL msg has been send
2024-09-15T20:17:45.879449822+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request
2024-09-15T20:17:45.881673996+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Response
2024-09-15T20:17:45.881911707+09:00 [INFO][SMF][CTX] [SMF] Add PSA success
2024-09-15T20:17:45.882250236+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:66239e21-c3f1-45d7-a235-da5196b30dad/modify |
```
The free5GC U-Plane (I-UPF) log when executed is as follows.
```
2024-09-15T20:17:45.856826976+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionEstablishmentRequest
2024-09-15T20:17:45.856857397+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] New session
2024-09-15T20:17:45.858045796+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:6 period:30000000000}]
2024-09-15T20:17:45.858506167+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:5 period:30000000000}]
2024-09-15T20:17:45.880417145+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-15T20:17:45.883070859+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-15T20:17:45.883669424+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:30000000000}]
2024-09-15T20:17:45.883955707+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:30000000000}]
2024-09-15T20:17:45.885715858+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
2024-09-15T20:17:45.886342331+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:30000000000}]
2024-09-15T20:17:45.886524176+09:00 [ERRO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] Mod CreateURR error: file exists
2024-09-15T20:17:45.886562714+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:30000000000}]
2024-09-15T20:17:45.886814514+09:00 [ERRO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] Mod CreateURR error: file exists
```
The free5GC U-Plane (PSA-UPF) log when executed is as follows.
```
2024-09-15T20:17:45.851961544+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.143:8805] handleSessionEstablishmentRequest
2024-09-15T20:17:45.851995844+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.143:8805][CPNodeID:192.168.0.141][CPSEID:0x2][UPSEID:0x1] New session
2024-09-15T20:17:45.853176719+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:3 period:30000000000}]
2024-09-15T20:17:45.853543116+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:4 period:30000000000}]
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
5: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::9c5d:5daf:d01a:e582/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_google"></a>

#### Ping google.com going through PSA-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (PSA-UPF).
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.222.46) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.222.46: icmp_seq=1 ttl=61 time=20.8 ms
64 bytes from 142.251.222.46: icmp_seq=2 ttl=61 time=19.6 ms
64 bytes from 142.251.222.46: icmp_seq=3 ttl=61 time=20.2 ms
```
The `tcpdump` log on U-Plane (PSA-UPF) is as follows.
```
20:24:38.118601 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1317, seq 1, length 64
20:24:38.137219 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1317, seq 1, length 64
20:24:39.119855 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1317, seq 2, length 64
20:24:39.136643 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1317, seq 2, length 64
20:24:40.121205 IP 10.60.0.1 > 142.251.222.46: ICMP echo request, id 1317, seq 3, length 64
20:24:40.137935 IP 142.251.222.46 > 10.60.0.1: ICMP echo reply, id 1317, seq 3, length 64
```
**Note. Make sure that the packets are not routed from I-UPF to the Internet.**

<a id="ping_8"></a>

#### Ping 8.8.8.8 going through I-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (I-UPF).
```
# ping 8.8.8.8 -I uesimtun0 -n
PING 8.8.8.8 (8.8.8.8) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=15.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=61 time=11.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=61 time=10.9 ms
```
The `tcpdump` log on U-Plane (I-UPF) is as follows.
```
20:25:30.285541 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1321, seq 1, length 64
20:25:30.298609 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1321, seq 1, length 64
20:25:31.287057 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1321, seq 2, length 64
20:25:31.296116 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1321, seq 2, length 64
20:25:32.288128 IP 10.60.0.1 > 8.8.8.8: ICMP echo request, id 1321, seq 3, length 64
20:25:32.296667 IP 8.8.8.8 > 10.60.0.1: ICMP echo reply, id 1321, seq 3, length 64
```
**Note. Make sure that the packets are not routed from PSA-UPF to the Internet.**

<a id="ping_docker"></a>

#### Ping 172.17.0.1 going through I-UPF

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane (I-UPF).
```
# ping 172.17.0.1 -I uesimtun0 -n
PING 172.17.0.1 (172.17.0.1) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=2.49 ms
64 bytes from 172.17.0.1: icmp_seq=2 ttl=64 time=2.27 ms
64 bytes from 172.17.0.1: icmp_seq=3 ttl=64 time=2.33 ms
```
The `tcpdump` log on U-Plane (I-UPF) is as follows.
```
20:26:12.583405 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1322, seq 1, length 64
20:26:12.583567 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1322, seq 1, length 64
20:26:13.583881 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1322, seq 2, length 64
20:26:13.583988 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1322, seq 2, length 64
20:26:14.584988 IP 10.60.0.1 > 172.17.0.1: ICMP echo request, id 1322, seq 3, length 64
20:26:14.585102 IP 172.17.0.1 > 10.60.0.1: ICMP echo reply, id 1322, seq 3, length 64
```
**Note. Make sure that the packets are not routed from PSA-UPF to anywhere.**

---
Using ULCL, I was able to confirm the very simple configuration for controlling the communication path to specific destination IP addresses.
I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.09.15] Updated to free5GC v3.4.3 (2024.09.12) and go-upf v1.2.1 (2023.12.19).
- [2022.08.16] Initial release.
