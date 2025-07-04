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


# Microsoft Teams: Automatic Work Location Detection (Office 365 Roadmap Feature 488800)

Microsoft is introducing **automatic work location detection** for Teams and Microsoft Places as part of the [Office 365 Roadmap (Feature 488800)](https://www.microsoft.com/en-gb/microsoft-365/roadmap?filters=&searchterms=488800). This new capability helps organisations keep users' work locations up to date, making it easier to find colleagues, book desks, and streamline hybrid work experiences. The feature is rolling out in 2025 and is designed to work seamlessly with Microsoft Places and Teams.

# What Is Automatic Work Location Detection?

Automatic work location detection allows Teams to update a user's work location based on two main signals:

- **Connection to a corporate wireless network (Wi-Fi SSID/BSSID)**
- **Connection to a desk peripheral (such as a monitor or docking station) configured in Microsoft Places**

When enabled, Teams can automatically set a user's location to "In the Office" or to a specific building, supporting features like desk booking, room reservations, and real-time presence in Microsoft Places.

# Prerequisites

Before enabling this feature, ensure the following:

- You are a Microsoft Teams administrator (for Teams policies) and an Exchange administrator (for Wi-Fi settings).
- Buildings and floors are configured in Microsoft Places.
- Desk pools or individual desk accounts are set up in Microsoft Places.
- Users must opt in and enable location sharing in their operating system and the Teams app. Admins cannot consent on behalf of users.

See [Management prerequisites](https://learn.microsoft.com/en-us/microsoft-365/places/managementoverview) for more details.

# How to Enable Automatic Work Location Detection

## 1. Enable the Work Location Detection Policy in Teams

First, create and assign a Teams work location detection policy. This policy enables the feature for selected users or groups.

```powershell
New-CsTeamsWorkLocationDetectionPolicy -Identity wld-enabled -EnableWorkLocationDetection $true
Grant-CsTeamsWorkLocationDetectionPolicy -PolicyName wld-enabled -Identity user@yourdomain.com
```

For more details, see the [official documentation](https://learn.microsoft.com/en-us/powershell/module/teams/new-csteamsworklocationdetectionpolicy).

## 2. Configure Detection via Wi-Fi (SSID/BSSID)

To detect when users are in the office based on Wi-Fi, configure the SSID and (optionally) BSSID lists in Microsoft Places:

**A. Set the SSID List:**
```powershell
Set-PlacesSettings -Collection Presence -WorkplaceWifiNetworkSSIDList 'Default:SSID-1;SSID-2'
```
This allows Teams to detect when a user is connected to a trusted office Wi-Fi network.

**B. Map BSSIDs to Buildings (for building-level accuracy):**
1. Create a CSV file with columns `BSSID` and `BuildingName`.
2. Map and upload the BSSID list:
```powershell
Add-WifiDevices -Action MapBuildings -InputFilePath test-file.csv
Add-WifiDevices -Action UploadEntries -InputFilePath test-file.csv -BuildingMappingFile mapping-file.csv
```
If only the SSID is configured, users' locations will show as "In the Office". With BSSID mapping, Teams can set the specific building.

**Note:** As of July 2025, Wi-Fi-based detection is in preview and will be widely available soon.

## 3. Enable Detection via Desk Peripherals (Microsoft Places)

You can also enable automatic location detection when users connect to a configured desk peripheral (like a monitor or docking station) at a bookable desk:

- Configure desk peripherals and associate them with desks in Microsoft Places. See [Configure desk peripherals](https://learn.microsoft.com/en-us/microsoft-365/places/configure-desk-peripherals).
- When a user plugs into a configured peripheral, Teams will automatically update their work location to "In the Office" or to the specific building.

Allow 24–48 hours for changes to propagate after configuration.

# User Consent and Experience

By default, users are opted out of automatic work location detection. When the policy is enabled, users are prompted to provide consent in the Teams desktop app (Windows or macOS). Admins cannot consent on behalf of users. Users must also enable location sharing in their OS and Teams app settings.

For more, see [Manage location sharing in Microsoft Teams](https://support.microsoft.com/office/manage-location-sharing-in-microsoft-teams-583dd649-87fc-4b23-aed6-f4e2279297f9#id0ebbd=windows).

# Best Practices and Next Steps

- Inventory your Teams Rooms and Teams Panel devices using the [Get Peripheral Information script](https://learn.microsoft.com/en-us/microsoftteams/rooms/get-peripheral-information).
- Ensure all office networks and desk peripherals are correctly configured in Microsoft Places.
- Remind users to enable location sharing and consent in Teams.
- For troubleshooting, allow up to 48 hours for new settings to take effect.

# Conclusion

Automatic work location detection in Microsoft Teams and Places is a powerful tool for hybrid workplaces. By leveraging Wi-Fi and desk peripherals, organisations can streamline desk booking, improve collaboration, and keep work locations up to date—making the modern office smarter and more connected.

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
Install-Module -Name MicrosoftPlaces -Force

# Connect to Microsoft Teams
Connect-MicrosoftTeams
```
---

### 2. Create a Work Location Detection Policy

```powershell
New-CsTeamsWorkLocationDetectionPolicy `
  -Identity "LondonHQ" `
  -EnableWorkLocationDetection $true
```
---

This example defines your London headquarters Policy. You can define multiple policies for different offices or remote work hubs and then assign them to users. This feels a little disjointed currently as this setting appears to only be an ON or OFF setting and then the configuration is done via MS Places 

```powershell
Set-PlacesSettings -Collection Presence -WorkplaceWifiNetworkSSIDList 'Default:MyWifi-SSID-Name'
```

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

