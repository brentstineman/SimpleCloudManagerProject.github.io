---
category: documentation
categories: ["docs", "screens"]
layout: document
permalink: /docs/screens/
title: "Screen Shots"
collection: "docs"
category: documentation
description: Wireframes and prototype screens shots with explanations.
---

As part of our early work, we have pulled together a few prototypes and screen shots to help illustrate the what we're going for. 

##Dashboard
When users log in, the first thing they will be presented with is their personal dashboard listing the resources that are assigned to them. This can optionally be the dashboard for any groups they are an administrator of. 

![](/images/screenshots/wireframe-dashboard.png)

The dashboard is where you can get a view of your resources and their usage against any quotas. You can also take "quick actions" like 'start', 'stop', or 'connect'. 

##Resource Detail View
You can also opt to drill down into a resource to see a detailed view of it. From here you can get more information on the resource and take actions that just don't fit into the "quick action" category. 

Another key aspect here is to see all the actions that have been taken on the resource, when they happened, and who did them.

![](/images/screenshots/wireframe-resourcedetails.png)

## Subscription Administration
As we called out in the [resources](/resources/) section, SCAMP needs to be able to automate away the provisioning of resources into various subscriptions. So the System Administrator(s) will need to be able to add subscriptions to the system. Here's a simple screen for that... 

![](/images/screenshots/wireframe-subscription.png)

Not much else to say here except you'll notice we're not using certificates. Going forward, Azure is leveraging its new "Resource Model", and as part of that, actions are taken using system administrators that have specific permissions to various resources. We want to embrace this new model and help highlight how it can be used. 

## Group Administration
Groups are a key concept for SCAMP. All resources are placed into groups for administration and quota tracking. So its important that the system administrators are able to list and define groups.

![](/images/screenshots/wireframe-grouplist.png)

Each group is assigned an owner and an optional quota. Furthermore, the system administrator or group owner can add resources to the group. These resources are in turn assigned out to individual owners and can have their own quotas set on them. 

![](/images/screenshots/wireframe-groupdetails.png)

## Early Prototypes
I'd also like to call out a key contribution to this effort by [Felix Rieseberg](http://www.felixrieseberg.com/about-me/ "About Felix"). He took these fairly crude wireframes and helped build out a prototype interface. 

![](/images/screenshots/prototype-resourcelist.png)
Our plan is to use [Ember](http://emberjs.com/ "Ember") for the front end, with the API's being called built in [ASP.NET 5](http://www.asp.net/vnext/overview/aspnet-vnext "ASP.NET 5"). Both are great new solutions that hope will highlight one of the team's personal objectives, and that's demonstrating the advances from both the Open Source community and Microsoft being used together to build some great solutions. 