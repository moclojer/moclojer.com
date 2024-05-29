---
title: 'Cloud Architecture'
date: 2024-05-28T00:00:00+07:00
intro_image: "images/illustrations/open-source.png"
intro_image_absolute: false
intro_image_hide_on_mobile: false
---

The [moclojer](https://github.com/moclojer/moclojer) is an Open Source product. To serve it as a service, we created an event-based architecture for event propagation and data processing, generating triggers for specific actions.

Below we have a diagram of the architecture we use for **moclojer cloud**:

* **back/api:** RESTful API used to persist data in the database and produce events in the message queue;
* **frontend:** web interface for user interaction;
* **yaml/generator:** unifies the `YAML` files of each *mock* into a single `YAML` file saved in the Object Store for `moclojer/foss` to consume;
* **cloud/ops:** responsible for monitoring and maintaining the infrastructure of domain, DNS, and other tools for host publication control;
* **[moclojer/foss](https://github.com/moclojer/moclojer):** moclojer open source extended as a framework with an isolated thread to sync the `YAML` from the Object Store;
* **[p001/proxy](https://github.com/moclojer/p001):** reverse proxy for the `moclojer/foss` service;

## Infrastructure

We use [DigitalOcean](https://m.do.co/c/70c384d0d807) as our infrastructure provider, our goal is to serve moclojer foss in a scalable way for all users. To achieve this, we use the [App Platform](https://www.digitalocean.com/products/app-platform/) to serve applications, [Managed Database](https://www.digitalocean.com/products/managed-databases) to provide managed databases (PostgreSQL and Redis), and [Droplet](https://www.digitalocean.com/products/droplets/) to serve the reverse proxy and [Supabase](https://supabase.com/auth) for frontend authentication.

Application logs are sent to [Logtail](https://marketplace.digitalocean.com/add-ons/logtail), installed via [DigitalOcean](https://m.do.co/c/70c384d0d807) add-ons.

### App Platform

The [App Platform](https://www.digitalocean.com/products/app-platform/) is a platform as a service (PaaS) that enables developers to build, deploy, and scale applications quickly and easily. It supports several programming languages, including Clojure 💜, and provides a range of features to help developers build and deploy applications more efficiently.

We have two applications on the App Platform:

**moclojer-app**

* `static_sites` serving the **frontend**, with `ingress` *(public access)*
* `services` serving the **back/api**, with `ingress` *(public access)*
* `workers` running the **yaml/generator** and **cloud/ops**
* `databases` running the **postgres** and **redis/mq** managed

**moclojer/foss**

* `services` serving the **moclojer/foss**, with `ingress` *(public access)* - with support for multiple domains

> We use managed infrastructure to focus our efforts on the product we serve as a service, not on the infrastructure that supports it. [DigitalOcean](https://m.do.co/c/70c384d0d807) helps us stay focused on what really matters.

## Diagram

{{< mermaid >}}
graph TD;

    subgraph "DigitalOcean"
        subgraph "database services"
            postgres
            redis/mq
        end

        subgraph "App Platform"
            back/api
            frontend
            cloud/ops
            yaml/generator
            moclojer/foss
        end

        subgraph "Droplet"
            p001/proxy
        end

        paas-service
        object-store
    end

    back/api --> postgres;
    back/api --> redis/mq;
    back/api --> object-store;
    frontend --> back/api;
    frontend --> supabase/auth;
    cloud/ops --> redis/mq;
    cloud/ops --> p001/proxy --> dns-service;
    cloud/ops --> paas-service;
    yaml/generator --> object-store;
    moclojer/foss --> object-store;
{{< /mermaid >}}

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%203.svg)](https://www.digitalocean.com/?refcode=70c384d0d807&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
