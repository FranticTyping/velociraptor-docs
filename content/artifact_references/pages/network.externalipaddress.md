---
title: Network.ExternalIpAddress
hidden: true
tags: [Client Artifact]
---

Detect the external ip address of the end point.

```yaml
name: Network.ExternalIpAddress
description: Detect the external ip address of the end point.
parameters:
  - name: externalUrl
    default: http://www.myexternalip.com/raw
    description: The URL of the external IP detection site.
sources:
  - precondition: SELECT * from info()
    query: |
        SELECT Content as IP from http_client(url=externalUrl)

```