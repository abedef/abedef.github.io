---
title: "Self-Hosting: Function-as-a-Service (FaaS) Offerings in 2023"
date: 2023-11-05T21:36:25-05:00
draft: false
---

# What is FaaS?

Function-as-a-Service (abbreviated FaaS) platforms offer developers the ability to quickly deploy their applications. Rather than deploy in the typical fashion involving a webserver and backend application, the developer can deploy each endpoint as an individual function, allowing developers to focus solely on their application.

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

Do note that I am not running my own Kubernetes cluster and I was led to believe that deploying a lot of these stacks would be much easier if you did. I had a frustrating time with a couple of these, to say the least.

## OpenWhisk

By far the most difficult to set up. The docker-compose method is not recommended for real deployments but is the easiest to get going. I couldn't find a way to get the service to bind its web UI and API to any of my public interfaces (always `172.17.0.1`, no matter what), which was a major annoyance. Googling around didn't return any useful results even though this seems to be one of the most widely supported FaaS offerings out there. I ended up resolving this issue by deploying a local NGINX instance on the same machine using [this configuration](https://gist.github.com/tluyben/b33548e1a3b8ed120312369fa67fb522) I found online.

It offers a web UI to write and test functions, and deploys each endpoint to a unique URL.

## OpenFaaS

Another solid contender with a healthy level of support. Also provides a web UI with a number of community-provided functions that can be deployed through the UI. These range from QR code generators to automated face blurring functions, accessible via HTTP requests.

## Fn Project

This was one of the easiest ones to set up and it got the job done fairly well. It provides a command line utility to bootstrap functions and to deploy them. Functions are grouped into applications, and each get their own endpoint.

## fx

This was also easy to set up but was not as feature rich. I did not spend much time working with this one but I disliked that it did not take a unified approach to its API. Support for each language was offered through some third party package integration. It gets the job done, but I wanted to keep trying other options in case I found something that was more mainstream and better supported.

One thing that makes fx unusable for me is that each service is by default exposed on its own port. In my case, it simplifies hosting greatly if each function is served from the same port, on separate endpoints.

## kNative

This was recommended on a few of the threads I was browsing for suggestions, but I did not get around to trying it out because it is made to run on Kubernetes. From what I understand it is commonly used in enterprise settings so you can expect *some* level of support.

## Dokku

Another one I did not get around to trying out since it did not fit my exact use case but I thought I should mention. Dokku's platform aims to replicate the same developer experience that Heroku provides.

## Coolify

I did not try this one either but similar to the above, this project offers a self-hostable Netlify/Vercel alternative.

# Conclusion

I really want to make OpenWhisk work for me but my deployment feels fragile and I get the feeling one day it will crash I will not be able to recover it. Until I have more time to put into this, I think I will be trying out Fn or OpenFaaS in my next projects.

I am currently working on a web-based game and am exploring writing a FaaS-based backend instead of my usual Docker containers or Systemd services. Watch this post for updates as I learn more about each tool.