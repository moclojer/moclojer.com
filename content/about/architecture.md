---
title: 'Cloud Architecture'
date: 2024-05-28T00:00:00+07:00
intro_image: "images/illustrations/open-source.png"
intro_image_absolute: false
intro_image_hide_on_mobile: false
---

The [moclojer](https://github.com/moclojer/moclojer) is an Open Source product. To serve it as a service, we created an event-based architecture for event propagation and data processing, generating triggers for specific actions.

## Backend Services

The following are the services and integrations that keep moclojer's backend alive:

* **back/api**: acts primarily as an API gateway for all client interactions, handling requests and routing them to appropriate services through message queues. Encapsulates initial business logic for user mocks, propagating necessary events to other services;
* **yaml/generator**: is responsible for updating every mock, as a processed and validated raw file, for any event, may it be creation, deletion, update, etc. It also manages a main raw mock file, which we call the `unified mock`, where all user mocks are aggregated, in order to be used in `moclojer/foss`'s runtime. The generated raw files are stored in our `object-store`;
* **cloud/ops**: manages and monitors our infrastructure, user mock domains and DNS records, by applying a facade on our two current used cloud solutions, namely DigitalOcean and CloudFlare;
* **[moclojer/foss](https://github.com/moclojer/moclojer)**: the open source version of moclojer, extended as a framework within an isolated service. Based on the content of the `unified mock` given from consequent events from `yaml/generator`, serves unique mock endpoints, acting therefore as the final API.

## Web Frontend

We currently serve a web interface for user interactions.

* **Stack**: ClojureScript, [Helix](https://github.com/lilactown/helix), [Refx](https://github.com/ferdinand-beyer/refx), TailwindCSS.

## Data Layer

* **Postgres**: We use Postgres as our main database.
* **Redis**: since moclojer is event-driven, we use Redis as a message broker, through its Pub/Sub mechanism.

## Communication and Integration

* **[p001/proxy](https://github.com/moclojer/p001):** a reverse proxy for our internal services that accesses outsider APIs;

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

**moclojer-foss**

* `services` serving the **moclojer/foss**, with `ingress` *(public access)* - with support for multiple domains

> We use managed infrastructure to focus our efforts on the product we serve as a service, not on the infrastructure that supports it. [DigitalOcean](https://m.do.co/c/70c384d0d807) helps us stay focused on what really matters.

### Infrastructure & Overall Architecture

This is a simplified overview of how our described services connect and interact between themselves after built, deployed and running on Digital Ocean.

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
    cloud/ops --> p001/proxy --> dns-services;
    cloud/ops --> paas-service;
    yaml/generator --> object-store;
    moclojer/foss --> object-store;
{{< /mermaid >}}

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%203.svg)](https://www.digitalocean.com/?refcode=70c384d0d807&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Overall Sequence Behaviour

This is an overview of how an optimistic run of the use case of a mock creation gets our services to work together.

{{< mermaid >}}
sequenceDiagram;

    participant frontend;
    participant back/api;
    participant db;
    participant yaml/gen;
    participant cloud/ops;
    box cloud providers
        participant cloudflare;
        participant digitalocean;
    end

    frontend ->>+ back/api: post /mocks;
    back/api ->>+ db: insert mock;
    db ->>- back/api: ok;
    back/api -->> yaml/gen: generate mock file;
    back/api ->>- frontend: 200 ok;

    loop mock offline
        frontend ->>+ back/api: get /mocks/123/publication;
        back/api ->>+ db: select mock publication status;
        db ->>- back/api: ok;
    end

    yaml/gen -->> yaml/gen: generated unified mock file;
    yaml/gen -->> cloud/ops: create domain;
    cloud/ops ->>+ cloudflare: create DNS record;
    cloudflare ->>- cloud/ops: 200 ok;
    cloud/ops ->>+ digitalocean: create domain;
    digitalocean ->>- cloud/ops: 200 ok;
    cloud/ops ->> back/api: mock published;
    back/api ->>+ db: update mock published;
    db ->>- back/api: ok
{{< /mermaid >}}

## Event Flow

As said before, we use Redis MQ in order to provide communication between our services, through Redis's Pub/Sub capabilities. Our queues are, separated by service, specifically:

* mock/api

** **domains.verification.dispatch**: a job at every 2 minutes that aggregates offline mocks to be verified, dispatching them to **`domain.verify`**;
** **unified.verification.dispatch**: a job at every 5 minutes that aggregates published mocks to be verified, dispatching them to **`unified.verify`**;
* **mock.publication**: sets a mock publication status;

* yaml/generator

** **mock.changed**: received when a mock content changes. It is saved to our `object-store`, and packaged to be unified;
** **mock.unified**: aggregates the new changed mock to the global `unified mock`, used thereafter by `moclojer/foss`;
** **mock.deleted**: removes given mock from our `object-store` and signals the recreation of the global `unified mock`;
** **unified.verify**: given the aggregated mocks, verifies if each of them is in teh `unified mock`, signaling the recreation if not;

* cloud/ops

** **domain.create**: interacts with DigitalOcean and CloudFlare, in order to setup the new domain host and DNS records;
** **domain.verify**: verifies the existence and validity of the newly created host, firing `mock.publication` thereafter;

* moclojer/foss

** **restart.mocks**: syncs to the `unified mock`, restarting the running moclojer server instance;
