# Unreachable Prefix Announcement (UPA)

Prefix summarization for SRv6 locators and UPA is configured on ASBR nodes between Core and Access ring 2, R011 and R12.

## A. Configuration

### 1. Routing policy
An routing policy is required to advertise routes between instances.
```cpp
(gl)[/configure policy-options policy-statement "export-locator-isis0"]
A:admin@R11-SR# info
    entry 10 {
        from {
            protocol {
                name [isis]
                instance 0
            }
        }
        to {
            protocol {
                name [isis]
                instance 1
            }
        }
        action {
            action-type accept
        }
    }
```

### 2. ISIS
ISIS configuration includes UPA announcement, ability to process and summarization.
```cpp
(gl)[/configure router "Base" isis 1]
A:admin@R11-SR# info
    admin-state enable
    advertise-router-capability as
    ipv6-routing native
    level-capability 1
    standard-multi-instance true
    traffic-engineering true
    export-policy ["export-locator-isis0"]
    area-address [49.1022.2222.22]
    multi-topology {
        ipv6-unicast true
    }
    loopfree-alternate {
        remote-lfa {
        }
        ti-lfa {
        }
    }
```
Configuration to be able to process UPA received
```cpp
    prefix-unreachable {
        process-received-upa true
        upa-lifetime 120
        upa-metric 4261412865
    }
```
```cpp
    segment-routing-v6 {
        admin-state enable
        micro-segment-locator "uSID-Alg0-Locator" {
            level-capability 1
            level 1 {
                metric 1
            }
        }
    }
    interface "system" {
        interface-type point-to-point
    }
    interface "toR12" {
        interface-type point-to-point
    }
    interface "toR13" {
        interface-type point-to-point
    }
    level 1 {
        wide-metrics-only true
    }
```
Summarization includes level and Algo0 to be announced as SRv6 locator
```cpp
    summary-address 2001:ac8::/32 {
        level-capability 1
        route-tag 21
        algorithm 0
        advertise-unreachable {
            advertise-route-tag 21
        }
    }
```

## B. Status

### 1. Normal status

ISIS LSP received on access node includes the summarization of prefix `2001:ac8::/32`

```cpp
A:admin@R16-IXR# /show router isis 1 database R11-SR.00-00 detail 

===============================================================================
Rtr Base ISIS Instance 1 Database (detail)
===============================================================================

Displaying Level 1 database
-------------------------------------------------------------------------------
LSP ID    : R11-SR.00-00                                Level     : L1 
Sequence  : 0xa2                   Checksum  : 0x8dbe   Lifetime  : 1086
Version   : 1                      Pkt Type  : 18       Pkt Ver   : 1
Attributes: L1                     Max Area  : 3        Alloc Len : 655
SYS ID    : 0100.0000.0011         SysID Len : 6        Used Len  : 655
 
TLVs : 
  Area Addresses:
    Area Address : (6) 49.1022.2222.22
  Supp Protocols:
    Protocols     : IPv4
    Protocols     : IPv6
  MT Topology:
    MT ID           : 0                        No Flags
    MT ID           : 2                        No Flags
  IS-Hostname   : R11-SR
  Router ID   :
    Router ID   : 10.0.0.11
  Router Cap : 10.0.0.11, D:0, S:0
    TE Node Cap : B E M  P
    SRv6 Cap: 0x0000
    SR Alg: metric based SPF
    Node MSD Cap: BMI : 0 SRH-MAX-SL : 10 SRH-MAX-END-POP : 9 SRH-MAX-H-ENCAPS : 7 SRH-MAX-END-D : 9
  Instance Id : 1 
    ITID          : 0
  I/F Addresses :
    I/F Address   : 10.0.0.11
  I/F Addresses IPv6 :
    IPv6 Address    : 3ffe::b
  TE IS Nbrs   :
    Nbr   : R12-SR.00                           
    Default Metric  : 10
    Sub TLV Len     : 97
    LclId    : 3
    RmtId    : 3
    MaxLink BW: 99999997 kbps
    Resvble BW: 99999997 kbps
    Unresvd BW: 
        BW[0] : 99999997 kbps
        BW[1] : 99999997 kbps
        BW[2] : 99999997 kbps
        BW[3] : 99999997 kbps
        BW[4] : 99999997 kbps
        BW[5] : 99999997 kbps
        BW[6] : 99999997 kbps
        BW[7] : 99999997 kbps
    Admin Grp : 0x0
    TE Metric : 10
    End.X-SID: 2001:ac8:b:e013:: flags:B algo:0 weight:0 endpoint:End.X-NXT-CSID-PSP lbLen:32 lnLen:16 funLen:16 argLen:64
  MT IS Nbrs     :
    MT ID           : 2                    
    Nbr   : R12-SR.00                           
    Default Metric  : 10
    Sub TLV Len     : 0
  TE IS Nbrs   :
    Nbr   : R13-IXR-a.00                        
    Default Metric  : 10
    Sub TLV Len     : 97
    LclId    : 4
    RmtId    : 3
    MaxLink BW: 99999997 kbps
    Resvble BW: 99999997 kbps
    Unresvd BW: 
        BW[0] : 99999997 kbps
        BW[1] : 99999997 kbps
        BW[2] : 99999997 kbps
        BW[3] : 99999997 kbps
        BW[4] : 99999997 kbps
        BW[5] : 99999997 kbps
        BW[6] : 99999997 kbps
        BW[7] : 99999997 kbps
    Admin Grp : 0x0
    TE Metric : 10
    End.X-SID: 2001:ac8:b:e016:: flags:B algo:0 weight:0 endpoint:End.X-NXT-CSID-PSP lbLen:32 lnLen:16 funLen:16 argLen:64
  MT IS Nbrs     :
    MT ID           : 2                    
    Nbr   : R13-IXR-a.00                        
    Default Metric  : 10
    Sub TLV Len     : 0
  TE IP Reach   :
    Default Metric  : 10
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.9
    Default Metric  : 0
    Control Info:    , prefLen 32
    Prefix   : 10.0.0.11
    Default Metric  : 30
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.5
    Default Metric  : 40
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.6
    Default Metric  : 20
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.7
    Default Metric  : 30
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.8
    Default Metric  : 20
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.10
  IPv6 Reach:
    Metric: ( IS) 1
    Prefix   : 2001:ac8::/32
    Sub TLV   :
      AdminTag(32bit): 21
    Metric: (DI ) 10
    Prefix   : 3ffe::9/128
    Metric: ( I ) 0
    Prefix   : 3ffe::b/128
    Metric: (DI ) 30
    Prefix   : 3ffe::5/128
    Metric: (DI ) 40
    Prefix   : 3ffe::6/128
    Metric: (DI ) 20
    Prefix   : 3ffe::7/128
    Metric: (DI ) 30
    Prefix   : 3ffe::8/128
    Metric: (DI ) 20
    Prefix   : 3ffe::a/128
  MT IPv6 Reach.  :
    MT ID           : 2                    
    Metric: ( I ) 0
    Prefix   : 3ffe::b/128
  SRv6 Locator  :
    MT ID : 0                    
    Metric: ( ) 1 Algo:0
    Prefix   : 2001:ac8::/32
    Sub TLV   :
      AdminTag(32bit): 21

Level (1) LSP Count : 1

Displaying Level 2 database
-------------------------------------------------------------------------------
Level (2) LSP Count : 0
-------------------------------------------------------------------------------
Control Info     : D = Prefix Leaked Down
                   S = Sub-TLVs Present
Attribute Flags  : N = Node Flag
                   R = Re-advertisement Flag
                   X = External Prefix Flag
                   E = Entropy Label Capability (ELC) Flag
Adj-SID Flags    : v4/v6 = IPv4 or IPv6 Address-Family
                   B = Backup Flag
                   V = Adj-SID carries a value
                   L = value/index has local significance
                   S = Set of Adjacencies
                   P = Persistently allocated
Prefix-SID Flags : R = Re-advertisement Flag
                   N = Node-SID Flag
                   nP = no penultimate hop POP
                   E = Explicit-Null Flag
                   V = Prefix-SID carries a value
                   L = value/index has local significance
Lbl-Binding Flags: v4/v6 = IPv4 or IPv6 Address-Family
                   M = Mirror Context Flag
                   S = SID/Label Binding flooding
                   D = Prefix Leaked Down
                   A = Attached Flag
SABM-flags Flags:  R = RSVP-TE
                   S = SR-TE
                   F = LFA
                   X = FLEX-ALGO
FAD-flags Flags:   M = Prefix Metric
===============================================================================
```

Locator for /32 installed in access node
```cpp
A:admin@R16-IXR# /show router isis 1 segment-routing-v6 micro-segment-locator 

===============================================================================
Rtr Base ISIS Instance 1 SRv6 Micro Segment Locator Table
===============================================================================
Prefix                             AdvRtr                        MT     Lvl/Typ
 AttributeFlags                     Tag                           Flags  Algo
-------------------------------------------------------------------------------
2001:ac8::/32                      R11-SR                        0      1/Int.
  -                                  21                            -      0
2001:ac8::/32                      R12-SR                        0      1/Int.
  -                                  21                            -      0
2001:ac9:d::/48                    R13-IXR-a                     0      1/Int.
  -                                  0                             -      0
2001:ac9:e::/48                    R14-IXR                       0      1/Int.
  -                                  0                             -      0
2001:ac9:f::/48                    R15-IXR-a                     0      1/Int.
  -                                  0                             -      0
2001:ac9:10::/48                   R16-IXR                       0      1/Int.
  -                                  0                             -      0
-------------------------------------------------------------------------------
No. of Micro Segment Locators: 6
-------------------------------------------------------------------------------
AttributeFlags: X    = External Prefix
                R    = Re-advertisement
                N    = Node
                E    = ELC
                A    = Anycast
Flags:          D    = Down
===============================================================================
```

With no UPA received

```cpp
A:admin@R16-IXR# /show router unreachable-route-table ipv6

===============================================================================
IPv6 Unreachable Route Table (Router: Base)
===============================================================================
Dest Prefix                                                  
  Proto                                     Age        Pref Metric
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
No. of Routes: 0
===============================================================================
```

### 2. Network issue

Simulating a failure on R08 by disabling uSID locator

```cpp
(gl)[/configure router "Base" segment-routing segment-routing-v6 micro-segment-locator "uSID-Alg0-Locator"]
A:admin@R08-SR# info
    admin-state disable
    block "SRv6-uSID-Alg0"
    un {
        value 8
    }
```
    
R11 starts announcing UPA
```cpp
A:admin@R11-SR# /show router isis 1 unreachable-routes originated ipv6-unicast 

===============================================================================
Rtr Base ISIS Instance 1 Unreachable Route Table (originated)
===============================================================================
Prefix                                        Algo Metric     Tag          Time
  SysID/Hostname                                MT   Lvl/Type   SpfVersion 
-------------------------------------------------------------------------------
2001:ac8:8::/48                               0    4261412865 21           106
  R11-SR                                        0    1/Int.     32            
-------------------------------------------------------------------------------
No. of Routes: 1
===============================================================================
```

R16 receives UPA
```cpp
A:admin@R16-IXR# /show router unreachable-route-table ipv6

===============================================================================
IPv6 Unreachable Route Table (Router: Base)
===============================================================================
Dest Prefix                                                  
  Proto                                     Age        Pref Metric
-------------------------------------------------------------------------------
2001:ac8:8::/48                             
 ISIS(1)                                     00h00m18s  15   4261412865
-------------------------------------------------------------------------------
No. of Routes: 1
===============================================================================
```

```cpp
A:admin@R16-IXR# /show router isis 1 segment-routing-v6 micro-segment-locator 

===============================================================================
Rtr Base ISIS Instance 1 SRv6 Micro Segment Locator Table
===============================================================================
Prefix                             AdvRtr                        MT     Lvl/Typ
 AttributeFlags                     Tag                           Flags  Algo
-------------------------------------------------------------------------------
2001:ac8::/32                      R11-SR                        0      1/Int.
  -                                  21                            -      0
2001:ac8::/32                      R12-SR                        0      1/Int.
  -                                  21                            -      0
2001:ac8:8::/48                    R11-SR                        0      1/Int.
  -                                  21                            D      0
2001:ac9:d::/48                    R13-IXR-a                     0      1/Int.
  -                                  0                             -      0
2001:ac9:e::/48                    R14-IXR                       0      1/Int.
  -                                  0                             -      0
2001:ac9:f::/48                    R15-IXR-a                     0      1/Int.
  -                                  0                             -      0
2001:ac9:10::/48                   R16-IXR                       0      1/Int.
  -                                  0                             -      0
-------------------------------------------------------------------------------
No. of Micro Segment Locators: 7
-------------------------------------------------------------------------------
AttributeFlags: X    = External Prefix
                R    = Re-advertisement
                N    = Node
                E    = ELC
                A    = Anycast
Flags:          D    = Down
===============================================================================
```

More detail in the LSP received from R11 for prefix `2001:ac8:8::/48`

```cpp
A:admin@R16-IXR# /show router isis 1 database R11-SR.00-00 detail             

===============================================================================
Rtr Base ISIS Instance 1 Database (detail)
===============================================================================

Displaying Level 1 database
-------------------------------------------------------------------------------
LSP ID    : R11-SR.00-00                                Level     : L1 
Sequence  : 0xa3                   Checksum  : 0xa9b5   Lifetime  : 1121
Version   : 1                      Pkt Type  : 18       Pkt Ver   : 1
Attributes: L1                     Max Area  : 3        Alloc Len : 722
SYS ID    : 0100.0000.0011         SysID Len : 6        Used Len  : 722
 
TLVs : 
  Area Addresses:
    Area Address : (6) 49.1022.2222.22
  Supp Protocols:
    Protocols     : IPv4
    Protocols     : IPv6
  MT Topology:
    MT ID           : 0                        No Flags
    MT ID           : 2                        No Flags
  IS-Hostname   : R11-SR
  Router ID   :
    Router ID   : 10.0.0.11
  Router Cap : 10.0.0.11, D:0, S:0
    TE Node Cap : B E M  P
    SRv6 Cap: 0x0000
    SR Alg: metric based SPF
    Node MSD Cap: BMI : 0 SRH-MAX-SL : 10 SRH-MAX-END-POP : 9 SRH-MAX-H-ENCAPS : 7 SRH-MAX-END-D : 9
  Instance Id : 1 
    ITID          : 0
  I/F Addresses :
    I/F Address   : 10.0.0.11
  I/F Addresses IPv6 :
    IPv6 Address    : 3ffe::b
  TE IS Nbrs   :
    Nbr   : R12-SR.00                           
    Default Metric  : 10
    Sub TLV Len     : 97
    LclId    : 3
    RmtId    : 3
    MaxLink BW: 99999997 kbps
    Resvble BW: 99999997 kbps
    Unresvd BW: 
        BW[0] : 99999997 kbps
        BW[1] : 99999997 kbps
        BW[2] : 99999997 kbps
        BW[3] : 99999997 kbps
        BW[4] : 99999997 kbps
        BW[5] : 99999997 kbps
        BW[6] : 99999997 kbps
        BW[7] : 99999997 kbps
    Admin Grp : 0x0
    TE Metric : 10
    End.X-SID: 2001:ac8:b:e013:: flags:B algo:0 weight:0 endpoint:End.X-NXT-CSID-PSP lbLen:32 lnLen:16 funLen:16 argLen:64
  MT IS Nbrs     :
    MT ID           : 2                    
    Nbr   : R12-SR.00                           
    Default Metric  : 10
    Sub TLV Len     : 0
  TE IS Nbrs   :
    Nbr   : R13-IXR-a.00                        
    Default Metric  : 10
    Sub TLV Len     : 97
    LclId    : 4
    RmtId    : 3
    MaxLink BW: 99999997 kbps
    Resvble BW: 99999997 kbps
    Unresvd BW: 
        BW[0] : 99999997 kbps
        BW[1] : 99999997 kbps
        BW[2] : 99999997 kbps
        BW[3] : 99999997 kbps
        BW[4] : 99999997 kbps
        BW[5] : 99999997 kbps
        BW[6] : 99999997 kbps
        BW[7] : 99999997 kbps
    Admin Grp : 0x0
    TE Metric : 10
    End.X-SID: 2001:ac8:b:e016:: flags:B algo:0 weight:0 endpoint:End.X-NXT-CSID-PSP lbLen:32 lnLen:16 funLen:16 argLen:64
  MT IS Nbrs     :
    MT ID           : 2                    
    Nbr   : R13-IXR-a.00                        
    Default Metric  : 10
    Sub TLV Len     : 0
  TE IP Reach   :
    Default Metric  : 10
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.9
    Default Metric  : 0
    Control Info:    , prefLen 32
    Prefix   : 10.0.0.11
    Default Metric  : 30
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.5
    Default Metric  : 40
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.6
    Default Metric  : 20
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.7
    Default Metric  : 30
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.8
    Default Metric  : 20
    Control Info: D  , prefLen 32
    Prefix   : 10.0.0.10
  IPv6 Reach:
    Metric: ( IS) 1
    Prefix   : 2001:ac8::/32
    Sub TLV   :
      AdminTag(32bit): 21
    Metric: (DI ) 10
    Prefix   : 3ffe::9/128
    Metric: ( I ) 0
    Prefix   : 3ffe::b/128
    Metric: (DIS) 4261412865
    Prefix   : 2001:ac8:8::/48
    Sub TLV   :
      AdminTag(32bit): 21
    Metric: (DI ) 30
    Prefix   : 3ffe::5/128
    Metric: (DI ) 40
    Prefix   : 3ffe::6/128
    Metric: (DI ) 20
    Prefix   : 3ffe::7/128
    Metric: (DI ) 30
    Prefix   : 3ffe::8/128
    Metric: (DI ) 20
    Prefix   : 3ffe::a/128
  MT IPv6 Reach.  :
    MT ID           : 2                    
    Metric: ( I ) 0
    Prefix   : 3ffe::b/128
  SRv6 Locator  :
    MT ID : 0                    
    Metric: ( ) 1 Algo:0
    Prefix   : 2001:ac8::/32
    Sub TLV   :
      AdminTag(32bit): 21
    Metric: (D) 4261412865 Algo:0
    Prefix   : 2001:ac8:8::/48
    Sub TLV   :
      AdminTag(32bit): 21
      End-SID   : 2001:ac8:8::, flags:0x0, endpoint:End-NXT-CSID-PSP, lbLen:32, lnLen:16, funLen:0, argLen:80

Level (1) LSP Count : 1

Displaying Level 2 database
-------------------------------------------------------------------------------
Level (2) LSP Count : 0
-------------------------------------------------------------------------------
Control Info     : D = Prefix Leaked Down
                   S = Sub-TLVs Present
Attribute Flags  : N = Node Flag
                   R = Re-advertisement Flag
                   X = External Prefix Flag
                   E = Entropy Label Capability (ELC) Flag
Adj-SID Flags    : v4/v6 = IPv4 or IPv6 Address-Family
                   B = Backup Flag
                   V = Adj-SID carries a value
                   L = value/index has local significance
                   S = Set of Adjacencies
                   P = Persistently allocated
Prefix-SID Flags : R = Re-advertisement Flag
                   N = Node-SID Flag
                   nP = no penultimate hop POP
                   E = Explicit-Null Flag
                   V = Prefix-SID carries a value
                   L = value/index has local significance
Lbl-Binding Flags: v4/v6 = IPv4 or IPv6 Address-Family
                   M = Mirror Context Flag
                   S = SID/Label Binding flooding
                   D = Prefix Leaked Down
                   A = Attached Flag
SABM-flags Flags:  R = RSVP-TE
                   S = SR-TE
                   F = LFA
                   X = FLEX-ALGO
FAD-flags Flags:   M = Prefix Metric
===============================================================================
```