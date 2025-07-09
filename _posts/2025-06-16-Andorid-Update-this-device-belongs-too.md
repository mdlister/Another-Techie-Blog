---
layout: post
comments: true
thumb: Android-This-Device-Belongs-to-Your-Organisation.jpg
smallthumb: intune
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



## Removing “This device belongs to your organisation name” from Android Lock Screens in Intune

While testing **Android device enrolment with Intune**, I noticed something odd: once a device was fully enrolled, its lock screen showed this message:

> **“This device belongs to your organisation name”**

At first, I assumed this was being pushed via a **Device Configuration Profile**, using the lock screen message setting:

![image.png](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/image-1411606f-9966-458c-8bc3-ed654aa706dc.png)

Turns out, that wasn’t it.

After digging into the docs, I came across this explanation:

> _"By default, the OEM default messages are shown. When you deploy a custom message using Intune, the Intune default message is also deployed. If you don't enter a custom message for the device's default language, then the Intune default message is automatically shown."_  
> [Corporate-owned Android Enterprise device restriction settings in Microsoft Intune | Microsoft Learn](https://learn.microsoft.com/en-us/intune/intune-service/configuration/device-restrictions-android-for-work?WT.mc_id=Portal-Microsoft_Intune_DeviceSettings)

Thinking it might be related to **Samsung Knox** (the OEM), I looked there next — but there were no options to edit or disable this message.

---

## The Real Reason This Message Appears

After speaking with Microsoft support, I learned this behaviour is **by design**.

When you connect Intune to **Managed Google Play**, you’re asked to provide an **organisation name**. If the device is enrolled as **Fully Managed**, Android displays that name on the lock screen. This isn’t something configurable through Intune or the OEM — it’s baked into the Android Enterprise setup.

Officially, there’s no Microsoft-supported way to remove or change this message after setup.

---

## Disclaimer About the Workaround

You *can* work around this behaviour — but it’s worth noting that Microsoft doesn’t recommend or support this approach. 

So proceed with caution.

---

## How to Remove the Lock Screen Message Anyway

This workaround involves **disconnecting** and **reconnecting** Managed Google Play, this time **without entering an organisation name** or in my example renaming it to something more generic.

---

### Step 1: Delete Your Organisation in Google Managed Play

Go to: [https://play.google.com/work/adminsettings](https://play.google.com/work/adminsettings)

Remove your organisation:

![Screenshot 1](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20102828-e4210e83-c4d6-4ac6-99ed-1e6db0c12920.png)  
![Screenshot 2](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20102838-5aa62784-5a73-42b2-8fc4-b472dacd486e.png)  
![Screenshot 3](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103551-f9747e8a-c6a8-4328-a384-ed196f3c966a.png)

---

### Step 2: Disconnect Google Managed Play from Intune


In Intune:  
**Tenant Administration > Under Tenant Status > Connector Status > Managed Google Play Connection > Managed Google Play**

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot 2025-07-09 131510.png")


Click **Disconnect**:

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103828-6a7ceda7-c8d6-433c-8e5e-d1878bae6d7f.png)

Confirm disconnection:

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103909-ea07f292-7172-43e8-bc5d-a15c717ded67.png)

> If you have devices enrolled as **COPE (Corporate-owned, personally enabled)** or **Fully Managed**, remove them first:

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20103949-f90aedc1-f36f-4d38-a87c-dccce2709b7c.png)  
![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104014-8ef599c1-997c-4795-808b-8d43701f418f.png)

You can batch remove devices to make it easier up to 100 devices. 

From the Devices view within Intune, show only Android devices, then in the menu select Bluk Actions

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot 2025-07-09 131824.png)

Then add the devices 

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot 2025-07-09 131852.png)

Once all the devices are removed you can unbind Google Play store from Intune. 

** Impact **

- Work profiles are removed  
- Fully managed devices will **factory reset**

![Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104242-b162bff1-90a2-4a60-b40e-a6103f73d764.png)

---

### Step 3: Reconnect Google Managed Play — Without a Name

Run through the setup wizard again:

![Wizard Start](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104256-28f846d8-aa4a-4db7-8106-97449f38b3e4.png)  

For me, I selected Sign up for Andorid Only, but select what ever you need for your organisation. 

![Select Android Only](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104333-7c709da1-3806-4449-beb6-13de7f74311d.png)  
![Get Started](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104346-6b36dddd-b954-4f7c-9035-0e684f3c4acb.png)

**Leave the organisation name blank** if you don't want it to appear on devices:

![Org Name Step](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104409-a51940a8-4607-4bd6-baff-cf664f6eb408.png)

Complete the wizard:

![GDPR](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104420-13aa7ddd-d2df-495d-a405-4f15d89fda9e.png)  
![Complete](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104502-88fdc1e9-2b01-46dc-bea1-5f1a917050b1.png)  
![Done](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/Screenshot%202025-06-13%20104523-c4ab1810-04bd-4d04-91b0-dad3be229849.png)

---

### Step 4: Create a New Android Enrolment Token

Go to:  
**Devices > Android > Enrolment > Corporate-owned, fully managed user devices**

Generate a new token:

![Token Screenshot](/assets/images/2025-06-16-Andorid-Update-this-device-belongs-too/image-0e75f655-3748-4064-9b67-6a0bef9b74f9.png)

> If you're using **Samsung Knox Mobile Enrolment**, update your configuration to use the new token.

---

### Step 5: Re-Approve Apps in Managed Google Play

The new connection wipes out your previous approvals. Re-add apps manually from the new Managed Google Play store.

---

## Final Thoughts

If your **Intune-enrolled Android devices** are showing the message _“This device belongs to your organisation name”_, and you didn’t configure it yourself — this is likely due to the default behaviour when linking to **Managed Google Play**.

While not officially supported, **resetting the connection and omitting the organisation name** gives you control over the lock screen message.

---

*Hope this helps someone! It wasn’t obvious at first — and definitely took some digging to figure out.*
