---
layout: document
categories: ["docs", "resources"]
permalink: /docs/features/resourceusage
title: "Resource Usage Monitoring"
collection: "docs"
category: documentation
share: false
description: A description of how we plan to track resource usage
---

> This is an incomplete **DRAFT**. Please use [this thread](https://github.com/SimpleCloudManagerProject/SCAMP/issues/150) for any feedback or questions regarding SCAMPs approach. 

Currently, the lowest level of cost containment you have in Azure is at the subscription level. And this solution works for a great many Azure customers. However, there are times when you want to place budget constraints or quotas on individual resources or groups of resources. While you can set up individual subscriptions for these groupings, after a while you can end up in a situation where subscription management becomes a challenge. 

This is one of the scenarios the Simple Cloud Manager Project, SCAMP, has set out to address. 

##Budgets##
Before we dive straight into budget, let’s cover the basic actors (user roles) involved:

- User is a member of a SCAMP resource group and can provision resources 
- Group Manager is a member of the group but can manage other members of the group
- Group Admin can create groups, configure groups they created, and assign group manager permissions
- System Admin – sets up subscriptions, resource templates, and assigns group administrator permissions

I’ve listed these in ascending order based on overall system permissions. Users have the lowest level of permissions in SCAMP, up to System Admins which have the greatest. Budgets however flow the other way. 

When the System Admin assigns group administrator permissions to someone, they also allocate that group admin a budget. As the group admin creates groups, they assign a portion of their budget out to those groups.  The group’s budget is in turn allocated out to all members of that group. The users, aka group members, then use this budget allocation to provision resources using the available templates.

Here’s a real world example… An instructor at an educational institution is going to be teaching a few classes this semester. The SCAMP system admin grants the instructor group admin permissions and assigns them a budget for the semester. The instructor then creates the groups they need for the classes, assigning a portion of their budget to each group. 

> **Backlog:** The current requirements call for groups to be optionally time bound. However, group admin budget allocations are not. This represents a gap in the design that will need to be addressed. 

When creating a group, a group administrator will define several attributes such as the name, what resource templates can be used, and when the group expires. They will also set the default budget allocation for each member of the group. When a user is added to the group, they will be given that budget amount by default. Altering the group’s default budget amount will not change the budget allocation of any users already assigned to the group. The group’s admin/managers will be able to alter the budget quotas for individual users at any time.
 
> **Backlog/JumpIn:** SCAMP v1.0 will not track if the total allocation to all users exceeds the budget allocation for the group. This would need to be checked as users are added to the group or user budget allocations are changed.
 
Please note that if a group budget allocation is reduced, only that portion of the group’s allocation that has not yet been used will be returned to the group admin for reallocation. This will prevent a group admin from re-using budget allocations by removing and recreating groups.

## Units, Templates, and Currency Rates ##
Measuring consumption in Azure is a bit complicated. Different resources are measured in different units of measurement and on different time frames. Azure Database (SQL) is billed by the day, DocumentDB collections are by hour, and Azure Compute (VMs) by the minute. Add into this mix and that we want SCAMP to eventually support globalization by allowing the System Admin to set the currency that should be used by the system and well… 

In looking at this situation, we decided that SCAMP itself would track resource consumption in terms of an arbitrary unit of measure that we’ll refer to as “consumption units”. Each resource deployed via SCAMP will be based on a template (more on that in another document). That template will define the resource’s “burn rate”. This is the number of units the resource will consume over a given span of time as well as the minimum time span measured. The system admin would then tell SCAMP what currency and how much of it each “unit” was worth. 

This approach grants SCAMP the flexibility to handle situations such as we discussed above with different types of resources being measured at different rates as well as time frames. 

Over time, as resource pricing changes and currency exchanges fluctuate, the system admin will be able to adjust the burn rate of resources and the current value of units to compensate. But ultimately, our hope that we’ll be able to leverage resource tagging and eventually an Azure provided billing API to measure usage. Until then this setup should work nicely. 

The downside is that we’ll need to track budget allocations and the consumption against them at the group admin, the group, and the user within a group level.

## Azure Web Apps vs Virtual Machines ##
Another aspect of consumption in Azure is if you are billed for individual resources, actions against a resource, a “container” in which resources exist. Examples of individual resources are items like DocumentDB collections and IaaS virtual machines. Operation based billing is items like Azure Storage and Service Bus Queues. Azure Web Apps is an example of the “container” based model. 

Unlike IaaS Virtual Machines, Azure Web apps are associated with a hosting plan. The charges are against the plan. This means that for SCAMP, we need to track the plan (to measure usage against), while also keep track of the web sites within the plan. 

Our plan is to handle this to have track one resource for the hosting plan. This resource would be automatically assigned to the group admin that created the group and visible to them in the SCAMP portal and it would have the appropriate burn rate as defined by the template it was created from. As users of the group create additional resources based on this template, they would be associated with the “hosting plan” resource, and have a zero burn rate. This parallels the way we believe group administrators will also be handling “shared” resources.

To go back to our earlier scenario of an instructor teaching a class, imagine if the class is on digital content publishing. The instructor would provision the web hosting plan as well as possibly a VM running MySQL for the students to attach their web sites too. The group administrator would be able to view/manage these shared resources since they “own” them, but the students would only be able to manage their individual web sites.  

## Resource Monitoring ##

*pending*