---
layout: post
comments: true
thumb: 2022-02-08-AVD-DAG-Limit-Increase/2022-02-08-AVD-DAG-Limit-Increase.png
title: Azure Virtual Desktop DAG Limit Increase
tagline: Azure Virtual Desktop (AVD) DAG Limit increase
slug: AVD-DAG-Limit-Increase
modified: 2022-02-08
description: AVD News update
tags:
  - AVD
  - Azure
  - Azure Virtual Desktop
  - WVD
  - Windows Virtual Desktop
---
Good News for those using Azure Virtual Desktop (AVD) for Desktop Application Group (DAG) limit has been increased. 

We've been working with AVD formally known as Windows Virtual Desktop (WVD) for over 2 years now and we haven't yet exceeded the DAG limit of 200 groups per tenant that was set back in December 2020 [https://docs.microsoft.com/en-us/azure/virtual-desktop/whats-new#application-group-limit-increase] https://docs.microsoft.com/en-us/azure/virtual-desktop/whats-new#application-group-limit-increase

Since then we have been looking at Windows 21h2 and AppAttach as a way to package up our applications to bring our last user communities over to AVD. 

We've had a few different thoughts about the approach for this, each office already has a DAG with the Desktop so we could package up each application in to it's own DAG and assign it to the hostpools for the users that require it but with the 200 max limit that meant that we'd be limited to hostpools and applications. 

We raised this with Microsoft who were happy to annouce that this would be coming in a few weeks and i'm glad to say it's here now! 

[https://docs.microsoft.com/en-us/azure/virtual-desktop/whats-new] https://docs.microsoft.com/en-us/azure/virtual-desktop/whats-new

We've decided to go with Team based DAG's packaging up all the software required for a team and assign that to the hostpool to reduce the number of hostpools required and hopefully be easier to manage and maintain in code. 