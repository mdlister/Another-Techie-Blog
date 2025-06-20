---
layout: post
comments: true
thumb: Microsoft.Graph_thumbnail.jpg
smallthumb: msgraph
title: Migrating away from MSOnline Powershell Module
tagline: Migrating away from MSOnline Powershell Module
slug: Migrating-away-from-MSOnline-Powershell-Module
modified: 2024-05-16
description: Embracing microsoft graph
tags:
  - Microsoft
  - Office365
  - Places
  - Teams
  - Modern Workplace
---

## Migrating away from MSOnline Powershell Module

Last week I was asked to help out with a problem with PowerShell where they were unable to extract the Licence assignements for users to determine if they were assigned via a group or directly. 

We were specifically looking at "ENTERPRISEPREMIUM" SKU. Licencing is an art within Microsoft and I'm no expert in this area but I do enjoy a challenge. 

I started with just exploring my own user details 

`Get-MgUserLicenseDetail -UserId Mike@anothertechieblog.co.uk`

Then got the SKU for what I was after. 

`Get-MgUserLicenseDetail -UserId Mike@anothertechieblog.co.uk | where {$_.skuid -eq "c7df2760-2c81-4ef7-b578-5b5392b571df"}`

Although this works, I didn't think it was very user friendly so a quick modification to the where clause 

`Get-MgUserLicenseDetail -UserId Mike@anothertechieblog.co.uk | where {$_.skuPartNumber -eq "ENTERPRISEPREMIUM"}`

`Get-MgUserLicenseDetail -UserId Mike@anothertechieblog.co.uk | where {$_.skuPartNumber -eq "ENTERPRISEPREMIUM"} | select -Expandproperty ServicePlans`

I was still a little stumped with this and jumped over to MSGraph Explorer as a useful tool to query MSGraph directly. [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)

Eventually after a little more googling I ended up with finding this on the documentation site. 

Get the properties and relationships of a group object. This operation returns by default only a subset of all the available properties, as noted in the Properties section. To get properties that aren't_ returned by default, specify them in a $select OData query option. The hasMembersWithLicenseErrors and isArchived properties are an exception and aren't returned in the $select query.

from the MS Documentation. I was able to confirm this worked in MSGraph Explorer 

https://graph.microsoft.com/v1.0/users/Mike@anothertechieblog.co.uk?$select=licenseAssignmentStates


In the end I settled for this:
```
$users = get-mguser -Property userprincipalname, licenseAssignmentStates | select userprincipalname -expandproperty licenseAssignmentStates 
$objectCollection=@()
foreach ($user in $users){
 
    if ($user.skuid -eq "c7df2760-2c81-4ef7-b578-5b5392b571df"){
    write-host "matched SKU" $user.userprincipalname
            $object = New-Object PSObject
            Add-Member -InputObject $object -MemberType NoteProperty -Name User -Value $user.userprincipalname
            if (!($user.AssignedByGroup))
                {
                Add-Member -InputObject $object -MemberType NoteProperty -Name AssignedByGroup -Value $false
                Add-Member -InputObject $object -MemberType NoteProperty -Name GroupID -Value ""
                Add-Member -InputObject $object -MemberType NoteProperty -Name GroupName -Value ""
                }
            else
                {
                Add-Member -InputObject $object -MemberType NoteProperty -Name AssignedByGroup -Value $true
                Add-Member -InputObject $object -MemberType NoteProperty -Name GroupID -Value $user.AssignedByGroup
                Add-Member -InputObject $object -MemberType NoteProperty -Name GroupName -Value (get-mggroup -GroupId ($user.AssignedByGroup) | select -expandproperty displayname)
                }
            $objectCollection += $object
        }
    }
$objectCollection
```