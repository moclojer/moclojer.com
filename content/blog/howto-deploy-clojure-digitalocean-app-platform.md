---
title: "Howto deploy Clojure DigitalOcean App Platform"
date: 2024-05-01T01:00:00+10:00
tags: ["digitalocean", "app-platform", "clojure", "docker", "tutorial"]
images: ["/blog/howto-deploy-clojure-digitalocean-app-platform.jpg"]
description: "Deploying a Clojure application on DigitalOcean's App Platform using Docker is easy and fast. Follow this tutorial to deploy your Clojure application."
---

![Howto deploy Clojure DigitalOcean App Platform](/blog/howto-deploy-clojure-digitalocean-app-platform.jpg?width=50%)

Developing an application in Clojure is fun and productive, but deploying the application can be a challenge for people who have not studied infrastructure (DevOps). DigitalOcean has a product called [App Platform](https://www.digitalocean.com/products/app-platform/), which is a PaaS (Platform as a Service), a platform that makes it easy to deploy applications in various languages, including Clojure.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%203.svg)](https://www.digitalocean.com/?refcode=70c384d0d807&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

In this tutorial, we will share how to deploy an application using Docker on DigitalOcean's App Platform. In this case, we will have a Docker image for Clojure - you can follow the same tutorial to deploy other languages by writing the Docker image.

## Using Docker

App Platform supports deploying applications in Docker containers. To deploy a Clojure application, you need to create a Dockerfile using the official Clojure Docker image (`docker.io/clojure:latest`), install your application's dependencies, and run the `.jar`.

To do this, simply create a file called `Dockerfile` at the root of the repository, which the App Platform will use to build and deploy the application.

**The example below shows how we do it at [moclojer](https://github.com/moclojer/moclojer)**.

```Dockerfile
FROM docker.io/clojure:temurin-21-tools-deps-alpine AS jar-build
RUN apk add git
WORKDIR /app
COPY . .
RUN clojure -M:dev --report stderr -m com.moclojer.build --uberjar
ENV PORT="8000"
ENV HOST="0.0.0.0"
ENV CONFIG="/app/moclojer.yml"
EXPOSE ${PORT}
VOLUME ${CONFIG}
ENTRYPOINT "java" "-jar" "/app/moclojer.jar"
```

## Creating a basic HTTP server in Clojure

> It might be a bit confusing, but don't worry

To simplify understanding, we will create a basic HTTP server in Clojure ([reitit](https://github.com/metosin/reitit) + [ring](https://github.com/ring-clojure/ring)) and deploy it on the App Platform - this way we can ensure that you understand how the whole process works.

```clojure
(ns sample-app.core
  (:gen-class)
  (:require [reitit.ring :as ring]
            [ring.adapter.jetty :as jetty]))

(def routers
  "ring router app"
  (ring/router
   ["/" {:get (fn [_]
                {:status 200, :body "http server running!"})}]
   ["/ping" {:get (fn [_]
                    {:status 200, :body "pong!"})}]))

(defn -main
  "software entry point"
  [& _]
  (jetty/run-jetty #'routers {:port 3000, :join? false})
  (println "server running in port 3000"))
```

We placed the project files in this repository [clojure-to-digitalocean-app-platform](https://github.com/moclojer/clojure-to-digitalocean-app-platform) for you to follow the tutorial.

## Creating the application on App Platform

### 1. Access the [App Platform](https://cloud.digitalocean.com/apps) and click on `Create App`

![create in app platform](/blog/howto-deploy-clojure-digitalocean-app-platform/step-1.png?width=75%)

### 2. Select the Github repository that you created with the Clojure project *(if you haven't done so, you can fork our [repository](https://github.com/moclojer/clojure-to-digitalocean-app-platform))*

![select repository in app platform](/blog/howto-deploy-clojure-digitalocean-app-platform/step-2.png?width=75%)

### 3. Select the project branch, we recommend `main` and the directory that contains the application *(if you work with a monorepo, this is where you select the application directory)*

![select branch/dir in app platform](/blog/howto-deploy-clojure-digitalocean-app-platform/step-3.png?width=75%)

### 4. Click `Next` and select the `Region` where the application will run

![select region in app platform](/blog/howto-deploy-clojure-digitalocean-app-platform/step-4.png?width=75%)

### 5. Review the information, click `Create Resources` and wait for the application to deploy

![review and create in app platform](/blog/howto-deploy-clojure-digitalocean-app-platform/step-5.png?width=75%)
![deploy in app platform done](/blog/howto-deploy-clojure-digitalocean-app-platform/done.png?width=75%)

### Done! Your Clojure application is running on DigitalOcean's App Platform

Here is the domain of your application running on the App Platform:

![link your in app platform done](/blog/howto-deploy-clojure-digitalocean-app-platform/link.png?width=75%)
