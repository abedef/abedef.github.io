---
title: "Self-Hosting: Function-as-a-Service (FaaS) Offerings in 2023"
date: 2023-11-05T21:36:25-05:00
draft: false
---

# What is FaaS?

Function-as-a-Service (abbreviated FaaS) platforms offers developers the ability to quickly deploy their applications. Rather than deploy in the typical fashion involving a webserver and backend application, the developer can deploy each endpoint as an individual function, allowing developers to focus solely on their application.

Some examples of FaaS are AWS Lambda and GCP Cloud Functions.

# Why is a FaaS useful?

FaaS platforms allow developers to deploy individual functions (usually as HTTP endpoints) in a simplified way. I have been handling my own projects' deployments manually for the past while, usually deploying to Docker or individual VMs. However, this can get tedious since I don't do a great job keeping track of my stack files for Docker or I forget to document a critical deployment detail. Most applications follow the same general steps to deployment, with minor differences.

Docker solves this issue partially, since a Docker-ready application will usually include the steps to build the Docker image and to deploy it, but I find that sometimes I forget another detail at this stage and deploy the image with a missing flag or forgotten port.

At the end of the day I want to be able to focus on quickly writing and deploying applications. To this end, I have been interested in deploying a FaaS platform that I can deploy my applications to.

# Self-hosting FaaS

If you're like me, you'd like to enjoy the conveniences of a FaaS platform but might be weary of using paid providers for personal projects. I feel comfortable moving fast and breaking things in my homelab, but once I open the gates to cloud providers invoicing me I start to get sweaty.

I found some time this past weekend to explore the different FaaS options available to us, and here is what I found.

* [OpenWhisk](https://github.com/apache/openwhisk) – Apache's highly scalable serverless platform
* [OpenFaaS](https://github.com/openfaas/faas)
* [Fn Project](https://fnproject.io/)
* [fx](https://github.com/metrue/fx) – Poor man's function as a service
* [kNative](https://knative.dev/docs/)
* [Dokku](https://dokku.com/) – An open source PAAS alternative to Heroku
* [Coolify](https://coolify.io/) – An open source & self-hostable Heroku / Netlify / Vercel alternative

