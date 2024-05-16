---
layout: post
comments: true
thumb: Screenshot 2024-05-16 at 23.29.40.png
smallthumb: Screenshot 2024-05-16 at 23.29.40.png
title: Introducing Microsoft Places
tagline: A first look at Microsoft Places
slug: Introducing-Microsoft-Places
modified: 2024-05-16
description: A first look at the new Microsoft Places feature
tags:
  - Microsoft
  - Office365
  - Places
  - Teams
  - Modern Workplace
---

## Exploring Microsoft Places: A New Era of Hybrid Workspaces

I've been waiting for over a year since I heard about Microsoft Places from a Microsoft Ignite video back in 2022. You can watch the announcement here: [Microsoft Ignite 2022](https://www.youtube.com/watch?v=ndrS8B89uO4). This week, I finally heard that it was in preview, and you could register your tenant for preview features. Since then, I've been trying to find out as much as I can about this new capability from Microsoft. Given its novelty, there's hardly any information out there on it.

If you aren't sure what Microsoft Places is, check out the [Introducing Microsoft Places article and video](https://www.microsoft.com/en-us/microsoft-365/blog/2022/10/12/introducing-microsoft-places-turn-your-spaces-into-places/).

From what I can tell so far from a few hours of tinkering, it's a massive enhancement on Microsoft Room Finder, with additional capabilities. Ignoring all the CoPilot functionality at the moment, as my work has no intentions of using it just yet, I'll try to focus on what it is, how it works, and how to configure it. After that, if I can get my personal tenant working with it, I'll explore the CoPilot side of it as well.

The rollout of Places seems to be staggered as mentioned [here](https://learn.microsoft.com/en-us/deployoffice/places/opt-in-places-preview). If you read through what capabilities are currently available, it's all about user location sharing:
- Getting the latest version of Outlook to support Places
- Enabling the web app experience for your users
- Accessing the Teams Preview environment for Places Features
- Configuring Places for your organization

The article goes on to say more features will be available like Desk booking and Places explorer, which will be really exciting as these are the features I'd want to try out in my company.

So there is an opt-in for additional features if you haven't yet [here](https://learn.microsoft.com/en-us/deployoffice/places/opt-in-places-preview#ready-to-go).

## Deploying Microsoft Places

I am guilty of skim reading documentation to find what I am looking for, and it took me a few reads to actually get this right, so I wanted to point it out as well.

The Deployment Guide has prerequisites that need to be followed, usually I skim over them, but I had to go back.

### Exchange
I'll talk about my experience and permissions so you know what worked, but that doesn't follow the best practices for security. I'm in the Organizational Management Role.

Thanks to parba from [AdminDroid Blog](https://blog.admindroid.com/how-to-deploy-microsoft-places-app/) for posting some more info on this one.

powershell
```
Get-ManagementRoleAssignment -Role TenantPlacesManagement -GetEffectiveUsers | Where {$_.EffectiveUserName -Eq "Adele Vance"}
```

I ran the role assignment to see which role it was linked to, and it was Org Management, so job done.

You need to be on the latest version of PowerShell (Installing PowerShell on Windows).

Verify you have one of the following subscriptions:

- Microsoft 365 Business Standard
- Microsoft 365 Business Premium
- Microsoft 365 or Office 365 (E1, E3, E5)
- Microsoft 365 or Office 365 (A1, A3, A5)

In a new PowerShell 7 window, run the following:


```
Install-Module -Name MicrosoftPlaces -AllowPrerelease -Force
```

I decided to enable it for the tenant and all users, but you can scope it to specific people if you want. Enable Places.


```
Connect-MicrosoftPlaces 
Set-PlacesSettings -Collection Places -EnablePlacesWebApp ‘Default:true'
```

Also worth noting this little note on the Microsoft Deployment Guide before you get too excited:

***It can take up to 1 day for users to gain access to the features.***

## So it's enabled, now what?
So from what I can see, the room finder within Outlook has been updated, and there is a new app within Teams called Places you can add, but I haven't found anywhere to add buildings, floors, rooms, or desks.

In my personal tenant, I don't have any room mailboxes or equipment mailboxes, so I will be creating a few of them to see if that makes a difference. I also don't have any Microsoft Teams Meeting Room licenses, as I think that will also play a part.

Stay tuned for more updates as I explore further features and capabilities of Microsoft Places!

### Here are some screenshots of What I found so far.

After enabling Places, I restarted teams and found the Places app availabile

![Screenshot 2024-05-16 at 12.50.59.png](/images/Screenshot%202024-05-16%20at%2012.50.59-b24c1e46-9183-41c6-82fb-b29620439314.png)

After adding the app, it's asking me to configure my working pattern, this can also be done in Outlook [Working Hours Microsoft Learn](https://support.microsoft.com/en-gb/office/set-your-work-hours-and-location-in-outlook-af2fddf9-249e-4710-9c95-5911edfd76f6#:~:text=Set%20work%20hours%20and%20location%20from%20Settings&text=Select%20Settings%20%3E%20Calendar.,days%2C%20times%2C%20and%20locations.)
![Screenshot 2024-05-16 at 12.51.19.png](/images/Screenshot%202024-05-16%20at%2012.51.19-4715f14b-0123-4a04-94c0-f24d12531615.png)
After this is the opportunity to update your location. 
![Screenshot 2024-05-16 at 12.51.51.png](/images/Screenshot%202024-05-16%20at%2012.51.51-c5417513-bfa5-41cc-be8e-ea47c57d7f7b.png)

![Screenshot 2024-05-16 at 12.52.04.png](/images/Screenshot%202024-05-16%20at%2012.52.04-ca95ad9a-d684-46cb-b22d-4fd3a20532ec.png)

Then the app opens, for me I could only see People across the top, in the videos and Microsoft screenshots there are more options, this could be because I haven't configured locations yet (this is also mentioned in the Outlook Link for working Hours above so I will be exploring that a little more in the next week) Or it could be as the deployment article from Microsoft suggests, those features are coming. 

![Screenshot 2024-05-16 at 12.52.35.png](/images/Screenshot%202024-05-16%20at%2012.52.35-38ada272-d790-4c11-8447-5a501f93db32.png)

Here is the Room Finder from Outlook Web 
![Screenshot 2024-05-16 at 13.00.30.png](/images/Screenshot%202024-05-16%20at%2013.00.30-01a3a443-542c-4f83-9b1b-0492f245b915.png)

When i queried the Powershell module to see what commands were available it didn't come back with many options but when trying to see how to use them there was no detailed help and nothing online as of yet. What I find when looking for these new things is everyone just rewrites the Microsoft articles, I wanted to be a bit different and try and talk through my experience of it. I hope this helped someone and thanks for reading. 