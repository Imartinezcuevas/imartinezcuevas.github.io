---
title: "Deployment models for cloud"
date: "2025-06-16"
summary: "A introduction to deployment models for cloud."
description: "Deployment models mini-essay."
toc: true
readTime: true
autonumber: true
math: false
tags: ["Cloud"]
showTags: false
---

# Deployment models
Deployment models indicate where the infrastructure resides, who owns and manages it, and how cloud resources and services are made available to user.

## Public cloud
In a public cloud model, users get access to servers, sotrage, network, security, and applications ans srevices delivered by cloud providers over the internet. Using web consoles and APIs, users can provision the resources ans service sthey need.
- Users don't own the servers hteir applications run or storage their data consumes.
- The user does not have any control over the computing environment and is subject to the preformance and security of thecloud provider's infrastructure.
- The cloud provider's pool of resources, including infrastructure, platforms, and software, are NOT dedicated for use by a single tenant of organization.

## Private cloud
Cloud infrastructure provisioned for exclusive use by a single organization. It may be owned, magaged, and operate by the organization, a third party or some combination of them.
A VPC is a public cloud offering that lets an organization establish its own private and secure cloud-like computing environment in a logically isolated part of a shared public cloud.
- A private cloud is a virtulized environment modeled to bring in the benefits of a public cloud platform without the perceived disadvantages of and open-end shared public platform.
  - The ability to leverage the value of cloud computing using systems that are directly managed or under perceived control of the organization's internal IT.
  - The ability to better utilize internal computing resources, sus as the organization's existen investments in hardware and software, thereby reducing costs.
  - Better scalability through virtualization and cloud bursting. That is, leveraging public cloud instances for a period of time, but returning to the private cloud when the surge is meet.
  - Controlled access and  graeter security measures customized to specific organizational needs.
  - The ability to expand and provision things in a relatively short amount of time, providing greater agility.   

## Hybrid cloud
Computing environment that connects an organization's on-premise private cloud and third-party public cloud, into a single flexible infrastructure for running the organizations applications and workloads. The mix of public and private cloud resources gives organizations the flexibility to choose the optimal cloud for each application or workload.
Workloads move freely between the two clouds as needs change. Hybrid cloud in interoperable, which means that the public and private cloud services can understand each other's APIs, configuration, data formats, and forms of authentication and authorization.

Two types of hybrid clouds:
- Hybrid mono cloud: only one cloud provider.
- Hybrid multi-cloud: open standards-based stack that can be deployed on any public cloud infrastructure.
