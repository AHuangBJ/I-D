---
title: General Source Address Validation Capabilities
abbrev: General SAV Capabilities
docname: draft-ietf-savnet-general-sav-capabilities-00
obsoletes:
updates:
date:
category: info
submissionType: IETF

ipr: trust200902
area: Routing
workgroup: savnet
keyword: Internet-Draft

author:
 -
  ins: M. Huang
  name: Mingqing Huang
  organization: Zhongguancun Laboratory 
  email: huangmq@zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: W. Cheng
  name: Weiqiang Cheng
  organization: China Mobile 
  email: chengweiqiang@chinamobile.com
  city: Beijing
  country: China
 -
  ins: D. Li
  name: Dan Li
  organization: Tsinghua University
  email: tolidan@tsinghua.edu.cn
  city: Beijing
  country: China
 -
  ins: N. Geng
  name: Nan Geng
  organization: Huawei Technologies 
  email: gengnan@huawei.com
  city: Beijing
  country: China
 -
  ins: L. Chen
  name: Li Chen
  organization: Zhongguancun Laboratory
  email: lichen@zgclab.edu.cn
  city: Beijing
  country: China

normative:

informative:
  RFC3704:
  RFC8704:
  RFC8955:
  RFC8956:
  I-D.ietf-sidrops-bar-sav:
  I-D.ietf-savnet-intra-domain-architecture:
  I-D.ietf-savnet-inter-domain-architecture:
...

--- abstract
The SAV rules of existing source address validation (SAV) mechanisms, are derived from other core data structures, e.g., FIB-based uRPF, which are not dedicatedly designed for source filtering. Therefore there are some limitations related to deployable scenarios and traffic handling policies.

To overcome these limitations, this document introduces the general SAV capabilities from data plane perspective. How to implement the capabilities and how to generate SAV rules are not in the scope of this document.

--- middle

# Introduction {#sec-intro}

Source address validation (SAV) can detect and prevent source address spoofing on the SAV-enabled routers. When a packet arrives at an interface of the router, the source address of the packet will be validated. The packets with unwanted source addresses or arriving at unwanted interfaces, will be considered invalid and usually be conducted elimination actions on. Only validated packets will continue to be handled or forwarded.

From the perspective of data plane validation, the SAV capabilities of existing mechanisms have two main limitations. One of them is the deployable scenario limitation. ACL rules can be configured for filtering unwanted source addresses at specific interfaces ([RFC3704]). However, ACL is not dedicatedly designed for source prefix filtering. There exist performance and scalability issues due to long-key based searching, and usually expert maintenance efforts are required. Strict uRPF and loose uRPF are two typical FIB-based SAV mechanisms ([RFC3704]) and are supported by most commercial routers/switches. FIB-based validation brings many benefits compared to ACL-based filtering but also induces some limitations. Strict uRPF is not applicable for asymmetric routing ([RFC8704]), which exists in various scenarios such as intra-domain multi-homing access, inter-domain interconnection, etc. Under asymmetric routing, a source prefix may have a different incoming interface from the next-hop interface of the matched entry, or the source prefix does not exist in the FIB at all. Loose mode can only block unannounced prefix, which results in massive false negatives. Overall, existing ACL-based or FIB-based SAVs can only be applied to specific scenarios and cannot be adaptive to various scenarios (e.g., symmetric vs asymmetric).

The other limitation is inflexible traffic handling policy. The current common practice is just to silently drop the spoofed packets. We don't know who benefits from this and who is the source. Further more, the clues of attacks are ignored, which could be very helpful for dealing with DDoS attacks etc.

The root cause of the above two limitations is that there is no tool specifically designed for source address filtering. That is, the capabilities of current tools are derived from other functions, e.g., FIB or ACL.

This document describes the general SAV capabilities that the data plane of SAV-enabled devices should have. Two kinds of capabilities will be introduced: validation mode and traffic handling policy. Validation modes describe how to apply validation in different scenarios. Traffic handling policies are the policies applied on the validated packets. By implementing the general SAV capabilities, the above two limitations of existing mechanisms can be overcome.

To achieve accurate and scalable source address validation, dedicated SAV rules are needed instead of just using those derived from other functions, e.g., FIB or ACL.

Note that the general SAV capabilities described in this document are decoupled with vendor implementation. Conforming implementations of this specification may differ, but the SAV outcomes SHOULD be equivalent to the described SAV capabilities. And also how to generate SAV rules is not the focus of this document.

## Terminology

Validation mode: The mode that describes the typical applications of SAV in a specific kind of scenarios. Different modes take effect in different scales and treat the validated packets differently.

SAV rule: The entry specifying the incoming interfaces of specific source addresses or source prefixes. The SAV rule expression might be slightly diffrent between validation modes.

Traffic handling policy: The policy taken on the 'invalid' packets validated by SAV.

## Requirements Language

{::boilerplate bcp14-tagged}

# Validation Modes {#sav-mode}
This section describes four validation modes (Mode1 in {{mode1}}, Mode2 in {{mode2}}, Mode3 in {{mode3}}, Mode4 in {{mode4}}). These modes take effect in different scales and need corresponding SAV rules to validate spoofing packets. By choosing modes in different scenarios appropriately, the network can be protected as much as possible while not impacting the forwarding of legitimate packets.  

## Mode 1: Interface-based prefix allowlist {#mode1}
Mode 1 is an interface-scale mode, which means it takes effect on a specific interface. The interface enabling Mode 1 is maintaining an interface-based prefix allowlist--SAV rule 1. Only the source prefixes recorded in the list will be considered valid, otherwise invalid.

Applying Mode 1 on an interface requires the complete knowledge of legitimate source prefixes connected to the interface. Mode 1 is suitable to the closed-connected interfaces such as those connecting to a subnet, a stub AS, or a customer cone. Such a mode can efficiently prevent the connected network from spoofing source prefixes of other networks.

Strict uRPF based on FIB belongs to this mode. However, to overcome the limitation of asymmetric routing, additional native source prefix-based SAV rule expression is suggested. This is essential for new SAV mechanisms or architectures such as EFP-uRPF [RFC8704], BAR-SAV {{?I-D.ietf-sidrops-bar-sav}}, Intra-domain/Inter-domain SAVNET ({{?I-D.ietf-savnet-intra-domain-architecture}}, {{?I-D.ietf-savnet-inter-domain-architecture}}), etc.

Mode 1 is the most efficient one if applicable, it is mutual exclusive with other modes. However, in some cases, it may be difficult for an interface getting all the legitimate source prefixes. If any legitimate prefix is not included in the allowlist, packets with this source addresses arriving at the interface may be improperly blocked. For example, the interface with a default route or the interface connecting to the Internet through a provider AS can hardly promise to know all the legitimate source prefixes. We need more modes to cover those scenarios.

## Mode 2: Interface-based prefix blocklist {#mode2}
Mode 2 is also an interface-scale mode, which means it takes effect on a configured interface. An interface cannot enable Mode 1 and Mode 2 at the same time. The interface enabling Mode 2 is maintaining an interface-based prefix blocklist--SAV rule 2. The source prefixes recorded in the list will be considered invalid, otherwise valid.

This mode does not require the complete knowledge of the illegitimate source prefixes on the interface, however some of them can be learned. Mode 2 is suitable for proactive filtering and reactive filtering. Usually the source prefixes that are sure to be invalid will be put into the blocklist, which is proactive filtering. Reactive filtering rules are usually installed in DDoS elimination for dropping packets with specific source addresses.

The prefix blocklist can be generated automatically, e.g., one of Intra-domain SAVNET architecture cases, blocking the incoming traffic with internal source prefixes on WAN interface. Or operators can configure the specific source prefixes to block from the interface. This is similar to ACL-based filtering, but more native SAV rule expression with better performance and scalability.

## Mode 3: Prefix-based interface allowlist {#mode3}
Mode 3 is a router-scale mode, which means it can validate traffic arriving at the router from all directions. The router enabling Mode 3 will record the protected source prefixes and maintain an interface allowlist for each source prefix--SAV rule 3. If a source prefix has an interface allowlist, the packet with this source prefix is considered valid only when its incoming interface is in the interface allowlist. Otherwise, the packet is considered invalid.

Applying Mode 3 in a router requires the complete knowledge of legitimate incoming interfaces for a specific source prefix. Mode 3 focuses on validating/protecting the interested source prefixes, it is applicable to the scenario where multiple interfaces are availalbe to provide potential connection to a(or a group) specific source prefix(es) , e.g. WAN-side open connected interfaces. Mode 3 provides a convenient and effective way to control which interface(s) is allowed to accept the specific source prefix, rather than to achieve similar effect by configuring Mode 2 on all other interfaces to block this source prefix.

Operators can configure the interface allowlist for a specific source prefix, to prevent DDoS attack related to this source prefix. Or the interface list for specific prefixes can be generated automatically, e.g., one capability defined by Inter-domain SAVNET architectures.

## Mode 4: Prefix-based interface blocklist {#mode4}
Mode 4 is also a router-scale mode, which means it can validate traffic arriving at the router from all directions. The router enabling Mode 4 will maintain an interface blocklist for a specific source prefix--SAV rule 4. If a source prefix has an interface blocklist, the packet with this source prefix is considered invalid when its incoming interface is in the interface blocklist. Otherwise, the packet is considered valid.

Applying Mode 4 in a router does not requires the complete knowledge of illegitimate incoming interfaces for a specific source prefix. Mode 4 focuses on prevent specific source prefix spoofing from specific directions, it is applicable to the scenario where multiple interfaces are facing specific source prefix spoofing attack, e.g. traffic coming in a network from open connected interfaces with its internal prefix as source address. Mode 4 provides a convenient and effective way to control which interface(s) should not accept the specific source prefix, rather than to achieve similar effect by configuring Mode 2 on each interface to block this source prefix, or Mode 4 for the specific source prefix but with a very long interface allowlist.

Operators can configure the interface blocklist for a specific source prefix, to prevent DDoS attack related to this source prefix. Or the interface list for specific prefixes can be generated automatically, e.g., one capability defined by Intra-domain SAVNET architectures.

## Validation Procedure
Mode 1 and Mode 2 are working on interface-level, they should not be enabled on same interface at same time. If they are enabled on same interface, Mode 2 must be ignored. Mode 3 and Mode 4 are working on router-level, they are also mutual exclusive with each othter, that is, they should not be enabled for a specific source prefix at same time. If so, Mode 4 must be ignored. Fruther more, Mode 1 are most preferred mode, which means while an interface has enabled Mode 1, the traffic for this interface don't need go through all other modes (Mode 2, Mode 3 or Mode 4) no matter whether they are configured. While the validation result on interface-level for Mode 2 is valid, the  traffic still need go through Mode 3 or Mode 4 if applicable. {{modes}} shows a comparison of different validation modes for dealing with source address validation.

~~~
  +--------------------------------------------------------------------+
  + Mode | Scale     | SAV rule                   | validation result  +
  +--------------------------------------------------------------------+
  + 1    | interface | 1: interface-based         | invalid if         +
  +      |           |    source prefix allowlist | not matched        +
  + 2    | interface | 2: interface-based         | invalid if         +
  +      |           |    source prefix blocklist | matched            +
  + 3    | router    | 3: prefix-based            | invalid if         +
  +      |           |    interface allowlist     | not matched        +
  + 4    | router    | 4: prefix-based            | invalid if         +
  +      |           |    interface blocklist     | matched            +
  +--------------------------------------------------------------------+
~~~
{: #modes title="A comparison of different validation modes"}

The general validation procedure is listed as below. The final validity state, either "valid" or "invalid",  will be returned after the procedure.

1) A packet arrives at the router, the source address and the incoming interface of the packet will be copied as the input for following validation process, and the initial validity state is set as 'valid'.

2) If Mode 1 is enabled on the incoming interface, the packet will be only validated based on SAV rule 1 (interface-base prefix allowlist), procedure returns with corresponding validity state. Otherwise--Mode 1 is not enabled on the incoming interface, perform following validation process.

3) If Mode 2 is enabled on the incoming interface, the packet will be validated based on SAV rule 2 (interface-base prefix blocklist). If validation result is invalid, precedure returns. If the validation result is valid or Mode 2 is not enabled, go through router-level validation procedures as below.

4) Similarly, if applicable, Mode 3 and Mode 4 validation procedure will be gone through based on SAV rule 3 (prefix-based interface allowlist) and SAV rule 4 (prefix-based interface blocklist) respectively, in which the precedure will return in case the validation result is invalid. For a specific source prefix, there should be only one router-level mode enabled.

# Traffic Handling Policies {#sec-policy}
After doing validation, the router gets the validity state of the incoming packet. For the packet with invalid state, traffic handling policies should be taken on the packet. Simply forwarding or silently dropping may not well satisfy the requirements of operators in different scenarios. This section suggests to provide flexible traffic handling policies to validated packets just like FlowSpec ([RFC8955], [RFC8956]).

The followings are the traffic control policies that can be taken. One and only one of the policies will be chosen for an "invalid" validation result.

- "Permit": Forward packets normally though the packets are considered invalid. This policy is useful when operators only want to monitor the status of source address spoofing in the network. The "Permit" policy can be taken together with the "Sample" policy.

- "Discard": Drop packets directly, which is the common choose of existing SAV mechanisms.

- "Rate limit": Enforce an upper bound of traffic rate (e.g., bps or pps) for mitigation of source address spoofing attacks. This policy is helpful while operators want do tentative filtering.

- "Traffic redirect": Redirect the packets to the specified points (e.g., scrubbing centers) in the network for attack elimination.

There are also traffic monitor policies that are optional. One of the useful traffic monitor policies is: 

- "Sample": Capture the packets with a configurable sampling rate and reports them to remote servers (e.g., security analysis center). The sampled packets can be used for threat awareness and further analysis. "Sample" can be taken together with any one of the above policies. Note that, existing techniques like NetStream or NetFlow can be used for "Sample". 


# Security Considerations
This document focuses on the general SAV capabilities. The generation of SAV rules is not discussed. There may be some security considerations for SAV generation, but it is not in the scope of this document. 

The "Sample" policy pushes data to remote servers. This function can be achieved by existing techniques like NetStream or NetFlow. The "Sample" policy may induce same security considerations as these techniques, and the corresponding documents have discussed them. 


# IANA Considerations {#sec-iana}

This document includes no request to IANA. 

# Acknowledgements

The authors appreciate the valuable comments from all members of the IETF SAVNET Working Group. We extend a special thanks to Aijun Wang for his guidance and suggestion. We extend a special thanks to Mingxing Liu and Changwang Lin for their feedback and contributions from vendor implemention point of view. 

# Contributors
-
  Mingxing Liu
  Huawei Technologies 
  China

  Email: liumingxing7@huawei.com
  
-
  Changwang Lin
  New H3C Technologies
  China

  email: linchangwang.04414@h3c.com
  
--- back

