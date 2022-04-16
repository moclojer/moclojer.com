---
title: "dynamic return"
date: 2018-11-18T12:33:46+10:00
draft: false
featured: true
weight: 6
---

The endpoint return does not have to be static, it can use dynamic data passed by **query string**, **URI** and _other parameters_.

Variables:

- `path-params`: the parameters passed to the endpoint `/hello/:username`
- `query-params`: the parameters passed in query string to the endpoint `?param1=value1¶m2=value2`
- `json-params`: the parameters passed in data request to the endpoint `{"param1": "value1"}`
