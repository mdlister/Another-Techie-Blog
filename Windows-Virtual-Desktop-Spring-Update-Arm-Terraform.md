# Windows Virtual Desktop Terraform Deployment
## WVD Spring Update
I wanted to put together some articles to explain how you can use Terraform and the AzureRM provider to build out a Windows Virtual Desktop deployment. 

I've been working with terraform since January 2021 as I can see the benefits that Infrastructure as Code (IAC) brings. I'm not a developer, I'd call myself a windows engineer with over 15 years supporting and looking after Microsoft products and mostly using VBScript and now Powershell to get everything I wanted to be done. 

I based a lot of this work on the guys over at [https://github.com/Azure/RDS-Templates](https://github.com/Azure/RDS-Templates) however I couldn't find anything online who was taking advantage of the WVD Spring Update where WVD is now fully integrated into AzureARM. 

I'll go through all the code and point out bits that I had issues with and some gotchas to help you out but you can find all the code in my github repo. [https://github.com/mdlister/Azure]https://github.com/mdlister/Azure

#The Code

