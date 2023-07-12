---
title: Demystifying Defender for Cloud
author: cotes
date: 2019-08-08 11:33:00 +0800
categories: [Blogging, Demo]
tags: [typography]
pin: true
math: true
mermaid: true
image:
  path: /commons/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

# Introduction

Defender for Cloud (DfC) is a cloud-native platform designed to protect multiple workloads, across several environments, by utilising a plethora of capabilities. Defender for Cloud isn’t new, it’s been available to the public since 2019, so why make a blog post on it now? 

Since the start of 2023 Defender for Cloud has cropped up multiple times during my day-to-day, and it seems to hold an air of mystery. During this article we will expand on why I think companies are (wisely) starting to pay more attention to defender for cloud, and dive a little deeper into the architecture. We will look at how all the pieces fit together in the Defender for Cloud puzzle and also how this materialises across multiple environments.

# The Rise in Popularity

This section talks about why I think DfC has seen rise in popularity during the start of this year. I think it’s important to highlight these points before we get into the meat of the product itself.

Firstly, changes to Microsoft’s licensing model has influenced a change which creates a need for servers to be onboarded to Defender for Servers, one of Defender for Clouds capabilities. 

Microsoft have now deprecated Microsoft Defender for Endpoint for Servers licensing model. This licensing model allowed organisations to onboard servers directly to Defender for Endpoint and remain compliant by purchasing a licence for the onboarded server. When the renewal date arrives, defender for endpoint for servers licensing will not be available to companies using this licensing. This means companies with more than sixty servers need to start utilising Azure-Arc and Defender for Servers.

<aside>
💬 Companies with less than 60 servers can use a licensing model called defender for business servers.

</aside>

The second point is that significant developments in recent releases of the Defender for Cloud connectors for AWS and GCP have made it much easier to connect and protect resources in multi-cloud environments.

# Architecture Overview

Part of the confusion around DfC I think is attributed to not necessarily complex architecture but a large one. Depending on the number of environments, onboarding can become overwhelming due to DfC’s requirement of connecting workloads in different environments via different methods. We will cover these in the next sections but let’s try and visualise the product’s coverage first.

![Architecture](/assets/img/EngineeringContent/DfCArchitecture.drawio-2.png)

As you can see Defender for Cloud provides security for a lot of different resources. In the sections below we will pick apart each how each of the environments are connected, but before we do that we need to pick apart Defender for Cloud itself. 

Defender for Cloud is like a wrapper, containing within it individual products to protect specific workload types. For example, virtual machines and other server like resources are protected by a capability within Defender for Cloud called Defender for Servers. On the other hand, containers onboarded to Defender for Cloud are protected by Defender for Containers. We can digest this easier by looking at the visual below.

![Structure](/assets/img/EngineeringContent/DFCStructure-2.jpg)

# Onboarding Methods

For different environments and workloads, onboarding differs slightly. Let’s assume for the sake of brevity that during this section we are talking about onboarding servers. 

## Direct Onboarding

Following recent feedback from the security community (and annoying during the writing of this post) Microsoft have now introduced direct onboarding. Previously you had to onboard non-azure resources to DfC via Azure Arc, a lightweight software agent which is capable of deploying various extensions to connected machines. Direct onboarding no longer makes Azure Arc a requirement. 

Direct onboarding allows you to deploy defender for endpoint agents via traditional defender for endpoint onboarding methods, but utilise the Defender for Cloud capabilities* and licensing capabilities without the need for Azure Arc. It’s a great and very well welcomed option for admins with limited time and resource.

*Microsoft does allude to some features of DfS Plan 2 not working correctly without Azure Arc. See the limitations listed in the documentation here.

## Azure Arc

For on-premises Microsoft recommends using Azure Arc, a service which essentially makes your on-premises workloads an Azure resource. Once these resources are joined to Azure, Arc enables simple provisioning of extensions for software such as Defender for Endpoint or the Azure Monitor Agent for log shipping. 

<aside>
💬 Azure Arc is free for onboarding purposes. Arc only becomes a paid service when you begin to manage configurations using Azure Arc. Guest configurations refer to compliance policies or operating system configurations.

</aside>

<aside>
⚠️ Check the Arc pricing before onboarding. At the time of writing connecting SQL instances with Azure Arc is the only paid service. But I wouldn’t be surprised if this changes.

</aside>

The onboarding process for Arc is different, and there are a tonne of guides out there showing the setup process, but to be honest the Azure Portal guides you through the setup fine. If you run into issues then I would always start troubleshooting the networking requirements first. 

## Third-party Cloud

Third-party cloud services can be connected to Defender for Cloud through the environments tab. They are straightforward to set up and require no real maintenance. However, there are a couple of things to note with third-party services:

1. They also use Azure Arc in the backend to provision the services, so the same network connectivity is required to the target machines in the third-party cloud. 
2. The extent of the coverage is not the same as Azure resources.

<aside>
⚠️ You can only protect Servers, Databases and containers in third party cloud systems.

</aside>