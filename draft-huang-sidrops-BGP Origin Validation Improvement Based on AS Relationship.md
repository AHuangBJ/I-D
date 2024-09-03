---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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

As the process of BGP Route Origin AS (ROA) objects registration and distribution in the RPKI system is independent with BGP prefix announcemnet in real network, therefore there always exist the accuracy and timeliness challenges of ROA records, which inevitably brings false positive issue of route origin validation. 有研究表明相当大比例的起源验证假阳性，存在起源冲突的2个AS之间存在一定的邻居关系。本文档通过引入AS邻居关系，帮助进一步排除ROV假阳性。 

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
