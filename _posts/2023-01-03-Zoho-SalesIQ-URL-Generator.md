---
layout: post
comments: true
thumb: zoho.png
smallthumb: zohosalesiq
title: Zoho SalesIQ URL Generator
tagline: Generate Zoho SalesIQ URL for your email Campaigns
slug: Zoho-SalesIQ-URL-Generator
modified: 2023-01-03
description: Generate Zoho SalesIQ URL's for your email Campaigns easily.
tags:
  - Zoho
  - SalesIQ
---

## Zoho SalesIQ URL Generator

Use the form below to generate a SalesIQ unique link for your email campaigns quickly. 

I've not been able to find anything online that will do this for you automatically or even from Zoho themselves which is surprising. 

Paste the URL you'd like the customer to click on in to the form, followed by there first and last name, and thier email address and click generate. 

This will return the unique SalesIQ url to use that will help track the user as they visit your website. 

URL:<input type="text" id=url><br>
First name:<input type="text" id="firstname"><br>
Second name:<input type="text" id="secondname"><br>
email:<input type="text" id=email><br>
<input type="button" id="addbutton" value="Generate URL" onClick="addTwoStrings()">
<br>
<br>
<label id="generatedurl">Click Generate</label>

<script>
function addTwoStrings()
{
var f1=document.getElementById("url").value;
var f2=document.getElementById("firstname").value;
var f3=document.getElementById("secondname").value;
var f4=document.getElementById("email").value;
var result=f1+"?siq_name="+f2+"%20"+f3+"&siq_email="+f4;
document.getElementById('generatedurl').innerHTML = result;
}
</script>
