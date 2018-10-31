# Securing Container Images

## Introduction

Previously, in our Docker Security Best Practices series, we took a deeper look into [Securing the Docker Host](https://anchore.com/blog/docker-security-best-practices-part-2/), and what best practices to follow. This post will continue the series, focusing on Docker images, the challenges that come with securing these artifacts, and what countermeasures can be taken to achieve a better container image security stance. Left out from this discussion will be any considerations that touch on host or runtime security. 

## Background on images

Simply put, a Docker image is a package that contains all requirements and metadata needed to run a Docker container. In essense, an image is a template from which a container can be instantiated. Images are immutatable, meaning, once they've been built, they cannot be changed. If someone were to make a change, a new image would be built as a result. 

Docker images are built in layers. A core compoment of images is known as the 'base layer'. This is the foundation upon which all other components/layers are added to. Commonly, base layers are minimal, and typically respresentative of common OSs.

Images are most often stored in a central location called a registry. From registries like Docker Hub, developers can store their own images, or find and download images that have already been created. 

## Continous approach

A very fundamental approach to countering threats for container images is automating building and testing. Organizations should set up the tooling to analyze images in a continous manner. In short, development teams need a structured and reliable process for building and testing the Docker images that are built. For container image specific pipelines, tools specifically designed to uncover vulnerabilites, configuration defects, and other security best practices, should be employed. Additionally, this tooling should give developers the option to create governance around the images being scanned. Meaning, based on configurable policy rules/gates, images can pass or fail the image scan step in the pipeline, and not be allowed to progress further. 

A simple example of how this might look:

1. Developer commits code changes to source control
2. CI platform builds container image
3. CI platform pushing container image to staging registry
4. CI platform calls a tool to scan the image
5. The tool passes or fails the images based on the policy mapped to the image
6. If the image passes the policy evaluation and any other tests defined in the pipeline, the image is pushed to a production registry.
 
 ## Image vulnerabilites

 As part of our continous approach, packages and components within the image should be scanned for common and known vulnerabilities. Image scanning should be able to uncover vulnerabilites contained within all layers of the image, not just the base layer. Moreover, image inspection and analysis should be able to detect vulnerabilies for OS and non-OS packages container within the images, as there are oftentimes vulnerable third-party libraries as part of application code. Should a new vulnerability for a package be published after the image has been scanned, the tool should be able to retrieve new vulnerability info for the applicable component, and alert the developers so remediation can begin.

 ### Policy 

 Organizations should be able to create and enforce policy rules based on the severity of the vulnerability as defined by the [Common Vulnerability Scoring System](https://www.first.org/cvss/).

 Example: If the image contains any vulnerable packages with a severity greater than medium, stop this build. 

 ## Image configuration

 Images should be configured to adhere with common best practices listed below. 

 ### Create a user for the container image

 Containers should be run as a non-root user, whenever possible. The `USER` instruction within the Dockerfile defines this. 

### Use trusted base images for container images

Ensure that the container image is based on another established and trusted base image downloaded over a secure channel. Docker images curated by the Docker community. For organizations, developers should be connecting and downloading images from secure, trusted, private registries. These trusted images should be selected from minimalistic technologies whenever possible to reduce attack surface areas. 

Docker Content Trust and Notary can be configured to give developers the ability to verify images tags and enforce client-side signing for data sent to and received from remote Docker registries. Content trust is disabled by default. 

For more info see [Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/) and [Notary](https://docs.docker.com/notary/getting_started/).

### Do not install unnecessary packages in the container

As stated above, images should be selected from minimalistic technologies whenever possible to reduce size and attack surface. Additionally, packages outside the scope and purpose of the container should not be installed. 


### Add the `HEALTHCHECK` instruction to the container image

The `HEALTHCHECK` instruction tells Docker how to test a container to check that it is still working. This instruction should be added to Dockerfiles. Based on the result of the healthcheck (unhealthy), docker could exit a non-working container and instatiate a new one. 

### Do not use update instructions alone in the Dockerfile

Do not use update instructions such as `apt-get update` alone or in a single line in the Dockerfile. Instead, run the following: 

```
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

This will help avoid duplication of packages and make updates easier. 

**Related** also see [leveraging the build cache](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache) for insight on how to reduce the number of layers and overall Dockerfile best practices.

### Use COPY instead of ADD when writing Dockerfiles

The `COPY` instruction copies files from the local host machine to the container file system. The `ADD` instruction can potentially retrieve files from remote URLs and perform unpacking operations. Since `ADD` could bring in files remotely, the risk of malicious packages and vulnerabilities from remote URLs is increased. 

### Do not store secrets in Dockerfiles

Do not store any secrets within container images. Developers may sometimes leave AWS keys, API keys, or other secrets inside of images. If attackers were do grab these secrets/keys they could be exploited. 

### Only install verified packages in containers

Only verified packages from trusted sources should be downloaded and installed. If you are downloading a package via `apt-get` from official Debian repositories, this is all set. To see how this can be verified within a Dockerfile see [Redis Dockerfile](https://github.com/docker-library/redis/blob/dc6dc737baa434528ce31948b22b4c6ccc78793a/5.0/Dockerfile).

## Implementation

One tool for implementing the best practices above is [Anchore](http://anchore.com). Anchore is a service that conducts static analysis on Docker images, and evaluates these images against user-defined checks. With Anchore, vulnerabilites within packages for OS and non-OS components can be identified, and the image configuration best practices described above can be enforced via policy rules. 