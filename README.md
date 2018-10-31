# Securing Container Images

## Introduction

Previously, in our Docker Security Best Practices series, we took a deeper look into [Securing the Docker Host](https://anchore.com/blog/docker-security-best-practices-part-2/), and what best practices to follow. This post will continue the series, focusing on Docker images, the challenges that come with securing these artifacts, and what countermeasures can be taken to achieve a better container image security stance.

## Images

Simply put, a Docker image is a package that contains all requirements and metadata needed to run a Docker container. In essense, an image is a template from which a container can be instantiated. Images are immutatable, meaning, once they've been built, they cannot be changed. If someone were to make a change, a new image would be built as a result. 

Docker images are built in layers. A core compoment of images is known as the 'base layer'. This is the foundation upon which all other components/layers are added to. Commonly, base layers are minimal, and typically respresentative of common OSs.

Images are most often stored in a central location called a registry. From registries like Docker Hub, developers can store their own images, or find and download images that have already been created. 
