# Securing Container Images

## Introduction

Previously, in our Docker Security Best Practices series, we took a deeper look into [Securing the Docker Host](https://anchore.com/blog/docker-security-best-practices-part-2/), and what best practices to follow. This post will continue the series, focusing on Docker images, the challenges that come with securing these artifacts, and what countermeasures can be taken to achieve a better container image security stance. Left out from this discussion will be any considerations that touch on host or runtime security. 

## Background on images

Simply put, a Docker image is a package that contains all requirements and metadata needed to run a Docker container. In essense, an image is a template from which a container can be instantiated. Images are immutatable, meaning, once they've been built, they cannot be changed. If someone were to make a change, a new image would be built as a result. 

Docker images are built in layers. A core compoment of images is known as the 'base layer'. This is the foundation upon which all other components/layers are added to. Commonly, base layers are minimal, and typically respresentative of common OSs.

Images are most often stored in a central location called a registry. From registries like Docker Hub, developers can store their own images, or find and download images that have already been created. 

## Continous approach

A very fundamental approach to countering threats for container images is automation. Organizations should set up the tooling to analyze images in a continous manner. In short, development teams need a structured and reliable process for building and testing the Docker images that are built. For container image specific pipelines, tools specifically designed to uncover vulnerabilites, configuration defects, and other security best practices, should be employed. Additionally, this tooling should give developers the option to create governance around the images being scanned. Meaning, based on configurable policy rules/gates, images can pass or fail the image scan step in the pipeline, and not be allowed to progress further. 

A simple example of how this might look:

1. Developer commits code changes to source control
2. CI platform builds container image
3. CI platform pushing container image to staging registry
4. CI platform calls a tool to scan the image
5. The tool passes or fails the images based on the policy mapped to the image
6. If the image passes the policy evaluation and any other tests defined in the pipeline, the image is pushed to a production registry.
 
 ## Image vulnerabilites

 As part of our continous approach