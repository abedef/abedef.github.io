---
title: "CI/CD Over the Years"
date: 2023-04-28T09:48:03-04:00
draft: true
---

I have been building CI/CD pipelines for my own projects ever since I encountered the concept at an internship. Most of the pipelines I worked with back then were configured in Jenkins, and were scripted using Jenkins' strange language choice of [Apache Groovy](http://www.groovy-lang.org).

In those years, there were not yet any standardized solutions built into version control providers like GitHub Actions and GitLab CI/CD. Pipelines commonly relied on an external build server that polled repositories for changes and acted on them. Nowadays, including a structured `yaml` file in the root of your repository is enough to get started.

Most (personal) projects don't really *need* a pipeline, but including one has several benefits:

* documentation of build steps
* visibility of latest build status
* ...

# GitLab CI/CD

# Types of deployments

```yaml
deploy-job:
  stage: deploy
  script:
    - rm -rf /usr/share/nginx/html/pharmacy/*
    - cp -r build/* /usr/share/nginx/html/pharmacy/
  tags:
    - shell
```

> Replace a directory with static build contents in it.
> Serve the contents with Nginx.

> Do a clean git clone into a directory.
> Run npm dev server from that directory.

> Replace a directory with (non-static) build contents.
> Serve the contents with node.

```yaml
notify:
  stage: notify
  tags:
    - shell
  script:
    - curl -s
      --form-string "token=$PUSHOVER_TOKEN"
      --form-string "user=$PUSHOVER_USER"
      --form-string "message=Finished deploying web-pharmacy"
      https://api.pushover.net/1/messages.json
```