---
title: "multi domain"
date: 2018-11-28T15:14:54+10:00
featured: true
draft: false
weight: 5
---

The moclojer specification supports multi-domain, within the same configuration file you can specify endpoints for multiple domains.

key: `host`

## Example

```yaml
- endpoint:
    host: moclojer.com
    method: GET
    path: /multihost
    response:
      status: 200
      headers:
        Content-Type: application/json
      body: >
        {
          "domain": "moclojer.com"
        }
- endpoint:
    host: sub.moclojer.com
    method: GET
    path: /multihost-sub
    response:
      status: 200
      headers:
        Content-Type: application/json
      body: >
        {
          "domain": "sub.moclojer.com"
        }
```
