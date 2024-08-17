---
title: "An Overview of 5G Networking"
bookCollapseSection: true
weight: 10
---

from 
https://datatracker.ietf.org/doc/draft-ietf-teas-5g-ns-ip-mpls/

### [B.1.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#appendix-B.1) [Key Building Blocks](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#name-key-building-blocks)

\[[TS-23.501](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId-3144)\] defines the Network Functions (UPF, Access and Mobility Function (AMF), etc.) that compose the 5G System (5GS) Architecture together with related interfaces (e.g., N1 and N2). This architecture has built-in control and user plane separation, and the control plane leverages a Service- Based Architecture (SBA). [Figure 33](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#_figure-28) outlines an example 5GS architecture with a subset of possible NFs and network interfaces.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#appendix-B.1-1)

  +-----+  +-----+  +-----+    +-----+  +-----+  +-----+ 
  |NSSF |  | NEF |  | NRF |    | PCF |  | UDM |  | AF  | 
  +--+--+  +--+--+  +--+--+    +--+--+  +--+--+  +--+--+ 
Nnssf|    Nnef|    Nnrf|      Npcf|    Nudm|        |Naf 
  ---+--------+--+-----+----------+---+----+--------+---- 
            Nausf|    Namf|       Nsmf| 
              +--+--+  +--+--+     +--+------+ 
              |AUSR |  | AMF |     |   SMF   | 
              +-----+  +--+--+     +--+------+ 
                       /  |           |      \\ 
Control Plane      N1 /   |N2         |N4     \\N4 
------------------------------------------------------------ 
User Plane          /     |           |         \\ 
                +---+  +--+--+  N3 +--+--+ N9 +-----+ N6  .---. 
                |UE +--+(R)AN+-----+ UPF +----+ UPF +----( DN  ) 
                +---+  +-----+     +-----+    +-----+     '---' 

[Figure 33](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#figure-33): [5GS Architecture and Service-based Interfaces](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-09#name-5gs-architecture-and-servic)




## [Appendix B.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B) [An Overview of 5G Networking](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-an-overview-of-5g-networkin)

This section provides a brief introduction to 5G mobile networking with a perspective on the Transport Network. This section does not intend to replace or define 3GPP architecture, instead its objective is to provide an overview for readers that do not have a mobile background. For more comprehensive information, refer to \[[TS-23.501](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId-3144)\].[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B-1)

### [B.1.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1) [Key Building Blocks](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-key-building-blocks)

\[[TS-23.501](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId-3144)\] defines the Network Functions (UPF, Access and Mobility Function (AMF), etc.) that compose the 5G System (5GS) Architecture together with related interfaces (e.g., N1 and N2). This architecture has built-in control and user plane separation, and the control plane leverages a Service- Based Architecture (SBA). [Figure 33](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-28) outlines an example 5GS architecture with a subset of possible NFs and network interfaces.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-1)

  +-----+  +-----+  +-----+    +-----+  +-----+  +-----+
  |NSSF |  | NEF |  | NRF |    | PCF |  | UDM |  | AF  |
  +--+--+  +--+--+  +--+--+    +--+--+  +--+--+  +--+--+
Nnssf|    Nnef|    Nnrf|      Npcf|    Nudm|        |Naf
  ---+--------+--+-----+----------+---+----+--------+----
            Nausf|    Namf|       Nsmf|
              +--+--+  +--+--+     +--+------+
              |AUSR |  | AMF |     |   SMF   |
              +-----+  +--+--+     +--+------+
                       /  |           |      \\
Control Plane      N1 /   |N2         |N4     \\N4
------------------------------------------------------------
User Plane          /     |           |         \\
                +---+  +--+--+  N3 +--+--+ N9 +-----+ N6  .---.
                |UE +--+(R)AN+-----+ UPF +----+ UPF +----( DN  )
                +---+  +-----+     +-----+    +-----+     '---'

[Figure 33](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-33): [5GS Architecture and Service-based Interfaces](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-5gs-architecture-and-servic)

Similar to previous versions of 3GPP mobile networks \[[RFC6459](https://www.rfc-editor.org/rfc/rfc6459)\], a 5G mobile network is split into the following four major domains ([Figure 34](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-29)):[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-3)

- UE, MS, MN, and Mobile:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.1.1)
    
    The terms User Equipment (UE), Mobile Station (MS), Mobile Node (MN), and mobile refer to the devices that are hosts with the ability to obtain Internet connectivity via a 3GPP network. An MS is comprised of a Terminal Equipment (TE) and a Mobile Terminal (MT).[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.1.2)
    
- Radio Access Network (RAN):[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.2.1)
    
    Provides wireless connectivity to UEs. A RAN is made up of the Antenna that transmits and receives signals to UEs and the Base Station that digitizes the signal and converts the Radio Frequency (RF) data stream to IP packets.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.2.2)
    
- Core Network (CN):[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.3.1)
    
    Controls the CP of the RAN and provides connectivity to the Data Network (e.g., the Internet or a private VPN). The Core Network hosts dozens of services such as authentication, phone registry, charging, access to Public Switched Telephony Network (PSTN) and handover.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.3.2)
    
- Transport Network (TN):[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.4.1)
    
    Provides connectivity between 5G NFs. The TN may provide connectivity from the RAN to the CN as well as within the RAN or within the CN. The traffic generated by NFs is - mostly - based on IP or Ethernet.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.1-4.4.2)
    

+----------------------------------------------+
|             +------------+    +------------+ |
| +----+      |            |    |            | |   .-------.
| | UE +------+    RAN     |    |     CN     +----(    DN   )
| +----+      |            |    |            | |   '-------'
|             +------+-----+    +------+-----+ |
|                    |                 |       |
|              +-----+-----------------+----+  |
|              |     Transport Network      |  |
|              +----------------------------+  |
|                                              |
|                    5G System                 |
+----------------------------------------------+

[Figure 34](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-34): [Building Blocks of 5G Architecture (A High-Level Representation)](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-building-blocks-of-5g-archi)

### [B.2.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2) [Core Network (CN)](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-core-network-cn)

The 5G Core Network (5GC) is made up of a set of NFs which fall into two main categories ([Figure 35](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-30)):[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-1)

- 5GC User Plane:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.1.1)
    
    The UPF is the interconnect point between the mobile infrastructure and the Data Network (DN). It interfaces with the RAN via the N3 interface by encapsulating/ decapsulating the user plane traffic in GTP tunnels (aka GTP-U or Mobile user plane).[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.1.2)
    
- 5GC Control Plane:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.2.1)
    
    The 5G control plane is made up of a comprehensive set of NFs. The description of these entities is out of the scope of this document. The following NFs and interfaces are worth mentioning, since their connectivity may rely on the Transport Network:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.2.2)
    
    - the AMF connects with the RAN control plane over the N2 interface[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.2.3.1.1)
        
    - the SMF controls the 5GC UPF via the N4 interface[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.2-2.2.3.2.1)
        

  +---------+    +-------------------------+
  |   RAN   |    |      5G Core (5GC)      |
  |         |    |                         |
  |         |    |   \[AUSF  NRF  UDM ...\]  |
  |         |    |         (SBA)           |
  |         |    |                         |
  |         | N2 |   +-----+ N11 +-----+   |
  |    CP -----------+ AMF +-----+ SMF |   |
  |         |    |   +-----+     +--+--+   |
  |         |    |                  |      |  Control Plane
-----------------------------------------------------------
  |         |    |                  | N4   |  User Plane
  |         | N3 |               +--+--+   | N6  .-------.
  |    UP -----------------------+ UPF +------->(   DN    )
  |         |    |               +-----+   |     \`-------'
  +---------+    +-------------------------+

[Figure 35](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-35): [5G Core Network (CN)](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-5g-core-network-cn)

### [B.3.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3) [Radio Access Network (RAN)](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-radio-access-network-ran)

The RAN connects cellular wireless devices to a mobile Core Network. The RAN is made up of three components, which form the Radio Base Station:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-1)

- The Baseband Unit (BBU) provides the interface between the Core Network and the Radio Network. It connects to the Radio Unit and is responsible for the baseband signal processing to packet.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-2.1.1)
    
- The Radio Unit (RU) is located close to the Antenna and controlled by the BBU. It converts the Baseband signal received from the BBU to a Radio frequency signal.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-2.2.1)
    
- The Antenna converts the electric signal received from the RU to radio waves[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-2.3.1)
    

The 5G RAN Base Station is called a gNodeB (gNB). It connects to the Core Network via the N3 (User Plane) and N2 (Control Plane) interfaces.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-3)

The 5G RAN architecture supports RAN disaggregation in various ways. Notably, the BBU can be split into a DU (Distributed Unit) for digital signal processing and a CU (Centralized Unit) for RAN Layer 3 processing. Furthermore, the CU can be itself split into Control Plane (CU-CP) and User Plane (CU-UP).[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-4)

[Figure 36](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-31) depicts a disaggregated RAN with NFs and interfaces.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.3-5)

            +---------------------------------+    +-----------+
            |                                 | N3 |           |
+----+  NR  |                                 +----+  5G Core  |
| UE +------+             gNodeB              |    |           |
+----+      |                                 +----+   (5GC)   |
            |                                 | N2 |           |
            +---------------------------------+    +-----------+
                            | |
                           .+ +.
                           \\   /
                            \\ /
            +---------------------------------+    +-----------+
            |           +-------------------+ |    |           |
            |           |                   | |    |           |
+----+  NR  | +----+ F2 |+----+ F1-U +-----+| | N3 |  +-----+  |
| UE +--------+ RU +-----+ DU +------+CU-UP+----------+ UPF |  |
+----+      | +----+    |+-+--+      +--+--+| |    |  +-----+  |
            |           |  |            |E1 | |    |           |
            |           |  | F1-C       |   | |    |           |
            |           |  |         +--+--+| | N2 |  +-----+  |
            |           |  +---------+CU-CP+----------+ AMF |  |
            |           |            +-----+| |    |  +-----+  |
            |           |     BBU split     | |    |  5G Core  |
            |           +-------------------+ |    |           |
            |       Disaggregated gNodeB      |    |           |
            +---------------------------------+    +-----------+

[Figure 36](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-36): [RAN Disaggregation](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-ran-disaggregation)

### [B.4.](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4) [Transport Network (TN)](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-transport-network-tn)

The 5G transport architecture defines three main segments for the Transport Network, which are commonly referred to as Fronthaul (FH), Midhaul (MH), and Backhaul (BH) \[[TR-GSTR-TN5G](https://www.itu.int/dms_pub/itu-t/opb/tut/T-TUT-HOME-2018-PDF-E.pdf)\]:[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-1)

- Fronthaul happens before the BBU processing. In 5G, this interface is based on eCPRI with Ethernet or IP encapsulation.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-2.1.1)
    
- Midhaul is optional: this segment is introduced in the BBU split presented in Appendix B.3, where Midhaul network refers to the DU- CU interconnection (i.e., F1 interface). At this level, all traffic is encapsulated in IP (signaling and user plane).[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-2.2.1)
    
- Backhaul happens after BBU processing. Therefore, it maps to the interconnection between the RAN and the CN. All traffic is encapsulated in IP.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-2.3.1)
    

[Figure 37](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-32) illustrates the different segments of the Transport Network with the relevant NFs.[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-3)

+---------------------------------------------------------+
|                    Transport Network                    |
|                                                         |
|    Fronthaul       Midhaul       Backhaul               |
|  +-----------+ +------------+ +-----------+             |
|  |           | |            | |           |             |
+--|-----------|-|------------|-|-----------|-------------+
 +-+--+      +-+-++         +-+-++        +-+---+     .---.
 | RU |      | DU |         | CU |        | UPF :----( DN  )
 +----+      +----+         +----+        +-----+     \`---'

[Figure 37](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-37): [5G Transport Segments](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-5g-transport-segments)

A given part of the transport network can carry several 5G transport segments concurrently, as outlined in [Figure 38](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#_figure-33). This is because different types of 5G NFs might be placed in the same location (e.g., the UPF from one slice might be placed in the same location as the CU-UP from another slice).[¶](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#appendix-B.4-5)

+---------+
|+----+   | Colocated
||RU-1|   | RU/DU
|+-+--+   |
|  | FH-1 |
|+-+--+   |
||DU-1|   |  +----+         +-----+         .---.
|+-+--+   |  |CU-1|         |UPF-1+--------( DN  )
+--|------+  +-+-++         +-+---+         \`---'
+--|-----------|-|------------|----------------------------+
|  |    MH-1   | |    BH-1    |          Transport Network |
|  +-----------+ +------------+                            |
|  +-----------+ +------------+ +-----------+              |
|  |    FH-2   | |    MH-2    | |    BH-2   |              |
+--|-----------|-|------------|-|-----------|--------------+
 +-+--+      +-+-++         +-+-++        +-+---+     .---.
 |RU-2|      |DU-2|         |CU-2|        |UPF-2+----( DN  )
 +----+      +----+         +----+        +-----+     \`---'

[Figure 38](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#figure-38): [Concurrent 5G Transport Segments](https://datatracker.ietf.org/doc/html/draft-ietf-teas-5g-ns-ip-mpls-08#name-concurrent-5g-transport-seg)

