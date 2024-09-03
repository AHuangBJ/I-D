---
title: "BGP Origin Validation Improvement Based on AS Relationship"
category: info

docname: draft-huang-sidrops-BGP Origin Validation Improvement Based on AS Relationship-00
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: ops
workgroup: sidrops
keyword:
 - origin validation
 - false positive
 - AS relationship

author:
 -
    fullname: Mingqing Huang
    organization: Zhongguancun Laboratory
    email: huangmq@zgclab.edu.cn

normative:

informative:


--- abstract

As the process of BGP Route Origin AS (ROA) objects registration and distribution in the RPKI system is independent with BGP prefix announcemnet in real network, therefore there always exist the accuracy and timeliness challenges of ROA records, which inevitably brings false positive issue of route origin validation. Studies have shown that, in most cases, two ASes related to origin validation false positive exist neigher AS relationship. 

This document aims to improve route origin validation by leveraging the AS relationshop information to identify the legitimate prefix conflicts as much as possible. 

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
