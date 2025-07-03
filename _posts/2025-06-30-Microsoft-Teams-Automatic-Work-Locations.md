---
layout: post
comments: true
thumb: teamsheader.png
smallthumb: teams
title: Microsoft Teams Automatically Set Work Locations Based on Organization’s Wi-Fi Network
tagline: Microsoft Teams Automatically Set Work Locations Based on Wi-Fi Network
slug: Microsoft-Teams-Automatic-Work-Locations
modified: 2025-07-03
description: "Step-by-step guide for setting up network-based location detection in Microsoft Teams to support Microsoft Places and desk booking."
tags:
  - Microsoft
  - Teams
  - PowerShell
  - Microsoft Places
---

# Setting Up Microsoft Teams Work Locations: A Practical Guide

As businesses move towards more flexible and hybrid ways of working, knowing where employees are physically located becomes increasingly important—not just for booking desks, but for streamlining IT operations and enhancing collaboration. Microsoft Teams now supports the ability to define work locations using network-based detection policies, making it easier for organisations to integrate desk booking and workspace management tools like Microsoft Places.

In this guide, we’ll walk you through how to configure Teams to recognise your employees’ locations using network identifiers. We’ll reference key Microsoft documentation and include a script to help you inventory your devices along the way.

---

## What Is a Work Location Detection Policy?

Microsoft has introduced a new PowerShell cmdlet: [`New-CsTeamsWorkLocationDetectionPolicy`](https://learn.microsoft.com/en-us/powershell/module/teams/new-csteamsworklocationdetectionpolicy?view=teams-ps). This allows administrators to define corporate network locations based on attributes like:

- **IP address ranges**
- **Trusted Wi-Fi SSIDs**
- **Network DNS suffixes**

When a user connects from one of these defined locations, Teams can automatically update their work location, allowing for features like desk check-in, status updates, and room bookings through Microsoft Places.

---

## Real-World Use Case: Desk Booking with Microsoft Places

Microsoft Places offers smart desk booking features. But for these to work smoothly, Teams needs to know when someone is in the office.

To do this, we must:

1. **Define your corporate locations** in Teams via PowerShell.  
2. **Inventory all your Teams Rooms and Teams Panel devices** to understand the peripherals and configurations.  
3. **Enable and manage booking policies** based on location and availability.

Step 2 is especially important for identifying devices that should be included in Places. Microsoft provides a script for this:  
[Get Peripheral Information](https://learn.microsoft.com/en-us/microsoftteams/rooms/get-peripheral-information)

---

## Step-by-Step: Setting Up Location Detection in Teams

Let’s break it down into actionable steps.

### 1. Open PowerShell and Connect to Teams

```powershell
# Install module if you haven't already
Install-Module -Name PowerShellGet -Force -AllowClobber
Install-Module -Name MicrosoftTeams -Force

# Connect to Microsoft Teams
Connect-MicrosoftTeams
```
---

### 2. Create a Work Location Detection Policy

```powershell
New-CsTeamsWorkLocationDetectionPolicy `
  -Identity "LondonHQ" `
  -IPRanges "192.168.0.0/24" `
  -WiFiSSIDs "LondonHQ-WiFi" `
  -NetworkDnsSuffixes "corp.contoso.com"
```
---

This example defines your London headquarters based on its IP range, Wi-Fi SSID, and DNS suffix. You can define multiple policies for different offices or remote work hubs.

---

### 3. Apply the Policy to Users or Groups

```powershell
Grant-CsTeamsWorkLocationDetectionPolicy -PolicyName "LondonHQ" -Identity user@yourdomain.com
```
For larger rollouts, consider applying the policy via groups or automation.

---

### 4. Inventory Your Teams Rooms Devices (for Microsoft Places)

Use the Microsoft-provided script to gather data on your Teams Rooms devices and peripherals:

[Download the script](https://learn.microsoft.com/en-us/microsoftteams/rooms/get-peripheral-information)

This will help ensure you're covering all required equipment before enabling desk booking in Microsoft Places.

---

### Final Thoughts

Setting up location detection in Microsoft Teams is a smart move for organisations embracing hybrid work. By leveraging Microsoft’s PowerShell tools and Microsoft Places integration, we can make the workplace experience smoother for both users and IT teams.

If you're unsure where to start, begin by identifying your office networks and running the inventory script on all Teams Room devices. From there, you'll be ready to build location-aware experiences that support collaboration, booking, and real-time presence.

