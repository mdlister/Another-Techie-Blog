---
layout: post
comments: true
thumb: Microsoft-Intune.jpg
smallthumb: Intune
title: Fixing "This Device Belongs to Your Organisation" Message on Intune Android Devices
tagline: Resolve unexpected lock screen footers on fully managed Intune Android Enterprise devices
slug: fix-intune-android-this-device-belongs-to
modified: 2025-06-16
description: Learn how to resolve the “This device belongs to your organisation” footer on Intune Android devices by resetting the Managed Google Play integration and reconnecting with a clean setup.
tags:
  - Intune
  - Android
  - Microsoft Endpoint Manager
  - Device Management
  - Mobile Device Management
  - MDM
  - Enterprise Mobility
  - Google Managed Play
---


While testing **Intune Android device enrolment**, I encountered something unexpected: once a device was fully enrolled, the lock screen displayed the message:

> **“This device belongs to your organisation name”**

At first, I assumed this was being set via **Device Configuration Profiles** in Intune, using the lock screen message setting:

![image.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/image-1411606f-9966-458c-8bc3-ed654aa706dc.png)

However, this wasn’t the cause. After diving deeper into Microsoft’s documentation, I discovered the following:

> _"By default, the OEM default messages are shown. When you deploy a custom message using Intune, the Intune default message is also deployed. If you don't enter a custom message for the device's default language, then the Intune default message is automatically shown."_  
> [Corporate-owned Android Enterprise device restriction settings in Microsoft Intune | Microsoft Learn](https://learn.microsoft.com/en-us/intune/intune-service/configuration/device-restrictions-android-for-work?WT.mc_id=Portal-Microsoft_Intune_DeviceSettings)

At this point, I suspected **Samsung Knox** (as the OEM) might be the source — but no available settings allowed me to control this message.

---

## ✅ How I Removed the “This device belongs to your organisation name” Lock Screen Message

The fix turned out to be deleting and recreating the **Managed Google Play connection** in Intune.

---

### Step 1: Remove the Organisation from Google Managed Play

Visit: [https://play.google.com/work/adminsettings](https://play.google.com/work/adminsettings)

Remove the organisation:

![Screenshot 2025-06-13 102828.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20102828-e4210e83-c4d6-4ac6-99ed-1e6db0c12920.png)  
![Screenshot 2025-06-13 102838.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20102838-5aa62784-5a73-42b2-8fc4-b472dacd486e.png)  
![Screenshot 2025-06-13 103551.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103551-f9747e8a-c6a8-4328-a384-ed196f3c966a.png)

Once the organisation is deleted, proceed to Intune.

---

### Step 2: Disconnect Google Managed Play from Intune

In Intune, go to:  
**Tenant Administration > Connectors and Tokens > Managed Google Play**

![Screenshot 2025-06-13 103828.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103828-6a7ceda7-c8d6-433c-8e5e-d1878bae6d7f.png)

Click **Disconnect** and confirm:

![Screenshot 2025-06-13 103909.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103909-ea07f292-7172-43e8-bc5d-a15c717ded67.png)

> ⚠️ If you have corporate-owned, personally enabled (COPE) Android devices enrolled, you must remove them first:

![Screenshot 2025-06-13 103949.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103949-f90aedc1-f36f-4d38-a87c-dccce2709b7c.png)  
![Screenshot 2025-06-13 104014.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104014-8ef599c1-997c-4795-808b-8d43701f418f.png)

Removing the work profile will wipe corporate data from the device.  
Fully managed devices will factory reset:

![Screenshot 2025-06-13 104242.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104242-b162bff1-90a2-4a60-b40e-a6103f73d764.png)

---

### Step 3: Reconnect Google Managed Play

With the connection removed, run through the setup wizard again:

![Screenshot 2025-06-13 104256.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104256-28f846d8-aa4a-4db7-8106-97449f38b3e4.png)

Choose **Android Only**:  
![Screenshot 2025-06-13 104333.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104333-7c709da1-3806-4449-beb6-13de7f74311d.png)

Click **Get Started**:  
![Screenshot 2025-06-13 104346.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104346-6b36dddd-b954-4f7c-9035-0e684f3c4acb.png)

Update your **organisation name** — this is the text that appears in *“This device belongs to your organisation name”* on the lock screen:

![Screenshot 2025-06-13 104409.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104409-a51940a8-4607-4bd6-baff-cf664f6eb408.png)

Enter your **data protection** info and agree to GDPR terms:

![Screenshot 2025-06-13 104420.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104420-13aa7ddd-d2df-495d-a405-4f15d89fda9e.png)

Click **Complete Setup**:

![Screenshot 2025-06-13 104502.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104502-88fdc1e9-2b01-46dc-bea1-5f1a917050b1.png)  
![Screenshot 2025-06-13 104523.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104523-c4ab1810-04bd-4d04-91b0-dad3be229849.png)

---

### Step 4: Create a New Android Enrolment Token

Go to:  
**Devices > Android > Enrolment > Corporate-owned, fully managed user devices**

Create a new token:

![image.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/image-0e75f655-3748-4064-9b67-6a0bef9b74f9.png)

> 🔁 If you're using Samsung Knox Mobile Enrolment, you'll need to add this new token there as well.

---

### Step 5: Re-approve Apps in Managed Google Play

Since the connection has changed, any previously approved apps will need to be **added again** from the new Managed Google Play store.

---

## Final Thoughts

If your **Android devices enrolled via Intune** are showing the message _“This device belongs to your organisation name”_, and it’s not set by a configuration profile, it’s likely due to the **default Intune message** shown when no custom language-specific message is defined.

By **resetting your Google Managed Play Store connection**, you can control and customise this message — or remove it entirely.

---

*Hope this helps someone out there — it wasn’t immediately clear to me!*
