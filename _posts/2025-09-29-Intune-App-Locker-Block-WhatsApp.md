---
layout: post
comments: true
thumb: applocker.png
smallthumb: intune
title: "Blocking WhatsApp on Windows 11 with AppLocker and Intune"
tagline: Practical App Control for IT Pros
description: "Learn how to block WhatsApp Desktop on Windows 11 using AppLocker and Intune. Step-by-step guide for IT engineers: identify the package family, create and deploy policies, and troubleshoot common issues."
slug: block-whatsapp-with-applocker-intune
tags: [intune, applocker, windows11, whatsapp, wdac, app-control, endpoint-security]
canonical_url: https://anothertechieblog.co.uk/block-whatsapp-with-applocker-intune
modified: 2025-09-29
---

> **Mission reminder:** Provide clear, actionable technical content for IT engineers working with Microsoft cloud, automation, and infrastructure as code.

# Blocking WhatsApp on Windows 11 with AppLocker and Intune

---

## Why AppLocker?

Disabling the Microsoft Store alone isn’t enough—users can still install WhatsApp via web links. AppLocker provides a robust solution by blocking the app based on its package family name (PFN), ensuring installation and execution are denied.

---

## Step-by-Step Guide

### Step 1 — Identify WhatsApp Package Family Name

On a test device with WhatsApp installed, run the following PowerShell command:

```powershell
Get-AppxPackage *WhatsApp* | Select Name, PackageFamilyName, PackageFullName | Format-List
```

this will output the exact package name and publisher you need for your AppLocker rule. WhatsApp's PFN is typically:

> 5319275A.WhatsAppDesktop

---

### Step 2 — Create an AppLocker Deny Rule

```markdown
## Step 2 — Create an AppLocker Deny Rule

**GUI Method (Recommended):**

1. Run `secpol.msc` → Application Control Policies → **AppLocker** → **Packaged app Rules**.
2. Right-click → **Create New Rule** → _Action_: **Deny** → _User or group_: **Everyone** (or a test group) → Next.
3. Choose **Use an installed packaged app as a reference**, select WhatsApp from the list, and finish the wizard.
4. In AppLocker properties, set **Packaged app rules** to **Audit only** for first-pass testing.

## Step 3 — Export and Prepare the AppLocker Policy for Intune

1. In the AppLocker MMC: right-click **AppLocker** → **Export Policy** → save as `AppLocker_WhatsApp.xml`.
2. Open the exported XML, copy the `<RuleCollection Type="Appx" ...> ... </RuleCollection>` block (keep the RuleCollection tags).
3. Save as UTF-8 text.

**Example XML Block:**

```xml
<RuleCollection Type="Appx" EnforcementMode="Deny">
  <FilePublisherRule Id="045d1335-3bd0-4baf-bddb-eb764593c334" Name="5319275A.WhatsAppDesktop, from WhatsApp Inc." Description="" UserOrGroupesktop" BinaryName="*">
        <BinaryVersionRange LowSection="*" HighSection="*" />
      </FilePublisherCondition>
    </Conditions>
  </FilePublisherRule>
</RuleCollection>
```
---

### Step 4 — Deploy via Intune

```markdown
## Step 4 — Deploy via Intune

1. In Intune portal: **Devices** → **Windows** → **Configuration profiles** → **Create profile**.
2. Platform: **Windows 10 and later**
3. Profile type: **Templates → Custom**
4. Add a setting:
   - **Name**: AppLocker - StoreApps - Deny WhatsApp
   - **OMA-URI**: `./Vendor/MSFT/AppLocker/ApplicationLaunchRestrictions/{GUID}/StoreApps/Policy` (replace `{GUID}` with a randomly generated GUID)
   - **Data type**: String
   - **Value**: Paste the `<RuleCollection Type="Appx">...</RuleCollection>` XML block (ensure UTF-8 encoding)
5. Assign to a small test device group. Save & create.

## Step 5 — Force, Monitor, and Test

- Wait for device to receive the profile (or run Sync from Company Portal).
- Verify enforcement: AppLocker may require a restart.
- Check **Event Viewer** → Applications and Services Logs → Microsoft → Windows → AppLocker for events (IDs 8022/8024 for packaged app allowed/blocked).


