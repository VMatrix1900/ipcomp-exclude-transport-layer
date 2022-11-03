---
title: "IP Payload Compression excluding transport layer"
abbrev: "IPComp excluding L4"
category: std

docname: draft-ls-6man-ipcomp-exclude-transport-layer-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "IPv6 Maintenance"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "IPv6 Maintenance"
  type: "Working Group"
  mail: "ipv6@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ipv6/"
  github: "VMatrix1900/ipcomp-exclude-transport-layer"
  latest: "https://VMatrix1900.github.io/ipcomp-exclude-transport-layer/draft-ls-6man-ipcomp-exclude-transport-layer.html"

author:
 -
  ins: C. Li
  name: Cheng Li
  organization: Huawei Technologies
  email: c.l@huawei.com
  country: China
 -
  ins: H. Shi
  name: Hang Shi
  organization: Huawei Technologies
  role: editor
  email: shihang9@huawei.com
  country: China

 -
  ins: M. Zhang
  name: Meng Zhang
  organization: Huawei Technologies
  email: zhangmeng6@huawei.com
  country: China

 -
  ins: X. Ding
  name: Xiaobo Ding
  organization: Huawei Technologies
  email: mirroryuri.ding@huawei.com
  country: China


normative:
  RFC2407:
  RFC3051:
  RFC3173:
  RFC8200:

informative:
  CPI-IANA:
    title: IPSEC IPCOMP Transform Identifiers
    author:
    date: 2022-10
    target: https://www.iana.org/assignments/isakmp-registry/isakmp-registry.xhtml#isakmp-registry-11
...

--- abstract

IP Payload Compression Protocol (IPComp) is used for compressing the IP payload in transmission to increase the communication performance. The IPComp is applied to payload of the IP datagram, starting with the first octet immediately after the IP header in IPv4, and starting with the first octet after the excluded IPv6 Extension headers. However, Transport layer information such as source port and destination port are useful in many network functions in transmission.

This document defines extensions of IP payload compression protocol (IPComp) to support compressing the payload excluding the transport layer information, to enable network functions using transport layer information (e.g., ECMP) working together with the payload compression. This document also defines an extension of IPComp to indicate the payload is not compressed to solve the out-of-order problems between the compressed and uncompressed packets.


--- middle

# Introduction

The IP Payload Compression Protocol (IPComp) {{RFC3173}} is defined to compress the IP payload in transmission in order to increase the communication performance between a pair of communicating nodes, provided the nodes have sufficient computation power and the communication is over slow or congested links.

In IP version 4, the compression is applied to the payload of the IP datagram, starting at the first octet following the IP header, and continuing through the last octet of the datagram. In the IPv6 context, IPComp is viewed as an end-to-end payload, and is not applied to IPv6 extension headers such as hop-by-hop, routing, and fragmentation extension headers{{RFC8200}}.  The compression is applied starting at the first IP Header Option field that does not carry information that must be examined and processed by nodes along a packet's delivery path, if such an IP Header Option field exists, and continues to the ULP payload of the IP datagram. Therefore, the transport layer information such as source port and destination port is compressed. When IPComp is used, the Next Header field of IP header is set to 108, IPComp Datagram. The IPComp header contains the original Next Header and the Compress Parameter Index(CPI) is inserted between the IP header and the compressed payload.

There are many network functions which needs the transport layer information to work. For example, flow-based ECMP, Carrier Grade Network Translation (CGNAT), Access Control List (ACL) may require source and destination port to identify the transport layer flow. Some Firewall (FW), Deep Packet Inspection (DPI) also need to inspect the transport layer information. If IPComp compressed those transport layer information, the nodes along the packet's delivery path can not obtain the source port and destination port. Therefore the IPComp is not compatible with the network functions requiring the transport layer information which makes it harder to deploy.

This document defines an extension of IPComp to support compressing the payload excluding the first 4 bytes of transport layer header which contains source port and destination port. In this way, the IPComp can coexist with many network functions which requires these information. This document also defines an extension to explicitly indicate the payload is uncompressed to solve the out-of-order processing between the compressed and uncompressed packets.

# Terminology

This document leverages the terms defined in {{RFC3173}}. The reader is assumed to be familiar with this terminology.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Problem Statement

Currently, the IPComp will compress all the IP payload which includes the transport layer information. If a layer 4 load balancer is deployed along the IPComp packet delivery path, then the load balancer can not obtain the source port and destination port to identify a flow without decompressing it first. In other words, the network functions which requires the transport layer information would also need to act as the decompression node of IPComp. This incompatibility makes the deployment of IPComp harder.


# Extensions to IPComp

This section defines two extensions of IPComp. The first extension is used to indicate the first four bytes of transport layer header which contains the source port and destination is excluded from the compression. The second extension indicates that the payload is not compressed.


## Four-bytes Exclusion Extension
This extension is used to indicate that the first four bytes of the transport layer header is excluded from the compression. The packet format using this extension is shown in {{IPComp-exclusion-layout}}(Demonstrated using IPv6 packet):

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version| Traffic Class |           Flow Label                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Payload Length        |  Next Header  |   Hop Limit   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                         Source Address                        +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                      Destination Address                      +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Next Header  |     Flags     |  Compression Parameter Index  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|       Source Port             |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
//                                                             //
//                       Compressed Payload                    //
//                                                             //
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #IPComp-exclusion-layout title="Packet format when using Four-bytes Exclusion Extension"}

To accomplish that there are two options to extend IPComp. The first option is to change the CPI field. Currently the CPI field identifies a particular compression algorithm. The defined CPI value can be found at {{CPI-IANA}}. We can define new CPI values to indicate the same compression algorithm with different compression range as shown in {{iana-CPI-table}}.

|Value|   Transform ID | References |
|:----|:---------------|:-----------|
| 0   | RESERVED       |{{RFC2407}}|
| 1   | IPCOMP_OUI     |{{RFC2407}}|
| 2   | IPCOMP_DEFLATE |{{RFC2407}}|
| 3   | IPCOMP_LZS     |{{RFC2407}}|
| 4   | IPCOMP_LZJH    |{{RFC3051}}|
| TBD | IPCOMP_OUI with four bytes exclusion    | This document |
| TBD | IPCOMP_DEFLATE with four bytes exclusion| This document |
| TBD | IPCOMP_LZS with four bytes exclusion    | This document |
| TBD | IPCOMP_LZJH with four bytes exclusion | This document |
{: #iana-CPI-table title="CPI with exclusion range Registry Entries"}

The second option is to change the Flags field. Currently, the Flags field is zero and ignored by the receiving node. We can introduce a bit to indicate whether the first four bytes is excluded from the compression range or not.

Which option is more suitable will be determined based on the discussion in the working group.

## Uncompressed Payload Extension

Currently, if the total size of a compressed payload and the IPComp header is not smaller than the size of the original payload, the IP datagram will be sent in the original non-compressed form without the IPComp header. In the receiving node, the packet with the IPComp header will go through the decompression co-processor first while the packet without the IPComp header will be forwarded directly. Going through different packet process path will cause the out-of-order of packets within the same flow, reducing the transport performance.

To solve the out-of-order packets within the same IPComp-enabled flow, we propose to add IPComp header no matter whether the packet within the IPComp-enabled flow is sent compressed or not. To indicate a packet is sent uncompressed, a new CPI value(TBD) is used. In this way, since all packets within the IPComp-enabled flow have IPComp header, they will go through the same process path and be processed in order. For uncompressed packet, the Next Header in the IPComp Header is copied into the Next Header in the IP header, and the IPComp Header is removed.


# IANA Considerations

TBD.

# Security Considerations

The security requirements and mechanisms described in {{RFC3173}} also apply to this document.

This document does not introduce any new security considerations.
