---
title: Linux.Proc.Arp
hidden: true
tags: [Client Artifact]
---

ARP table via /proc/net/arp.

```yaml
name: Linux.Proc.Arp
description: ARP table via /proc/net/arp.
parameters:
  - name: ProcNetArp
    default: /proc/net/arp
sources:
  - precondition: |
      SELECT OS From info() where OS = 'linux'

    query: |
        SELECT * from split_records(
           filenames=ProcNetArp,
           regex='\\s{3,20}',
           first_row_is_headers=true)

```
