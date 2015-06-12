---
layout: document
categories: ["docs", "resources"]
permalink: /docs/features/resourceusage/
title: "Resource Usage Monitoring"
collection: "docs"
category: documentation
share: false
description: A description of how we plan to track resource usage
---

> Please use [this thread](https://github.com/SimpleCloudManagerProject/SCAMP/issues/150) for any feedback or questions regarding SCAMPs approach. 

Currently, the lowest level of cost containment you have in Azure is at the subscription level and this works for a great many Azure customers. However, there are times when you want to place budget constraints or quotas on individual resources or groups of resources. While you can set up individual subscriptions for these groupings, after a while you can end up in a situation where subscription management becomes a challenge. 
This is one of the scenarios the Simple Cloud Manager Project, SCAMP, has set out to address.
 
## Budgets ##
Let’s review the basic actors (user roles) involved:

- **User** is a member of a SCAMP resource group and can provision resources 
- **Group Manager** is a member of the group that add/manage other users to the group
- **Group Admin** can create groups, configure groups they created, and assign group manager permissions
- **System Admin** sets up subscriptions, resource templates, and assigns group administrator permissions

I’ve listed these in ascending order based on overall system permissions. Users have the lowest level of permissions in SCAMP, up to System Admins which have the greatest. Budgets however flow the other way.
 
When the System Admin assigns group administrator permissions to someone, they also allocate that group admin a budget. As the group admin creates groups, they assign a portion of their budget out to those groups.  The group’s budget is in turn allocated out to all members of that group. The users then use their budget allocation to provision resources using the available SCAMP resource templates (a topic for another day).

In addition to an amount, a budget also has a budget period (week, month, day, etc…), and start & end dates. For example, an Azure Subscription has a monthly billing period which may start on the first day of a given month and be a 6 month or annual commitment (ending date). A project may decide to track usage by sprint (resulting in budget periods that may only be 1-2 weeks). Or an educational institution may use a budget period that’s closer to length of a semester. 

These secondary attributes of the budget (budget period, start date, end date) will be set initially by the system administrator when a group admin is appointed. These settings will then be inherited by the groups and users that are able to use portions of that budget.


> **Backlog:** SCAMP may eventually support multiple budgets per group admin and/or the ability for group admins to “share” a single budget. Thus allowing group administrators to select which budget a group is allocated funding from. 

**Example A:**

> An instructor at an educational institution is going to be teaching a few classes this semester. The SCAMP system admin grants the instructor group admin permissions and assigns the instructor a budget for the semester (a single amount with a start and end date that align to the semester). The instructor then creates the groups they need for the classes, assigning a portion of their budget to each group. The budget itself has a single billing period (the entire duration), and a fixed start/end date. 

**Example B:**

> A project manager has been allocated a monthly recurring budget for 2 projects for the next 6 months. The instructor gives 60% of that budget to one project, and 40% to the other. Each user in these groups can use 10% of that budget. In this case the budget is monthly, so at the end of each month, all groups/users reset to 0. 

When creating a group, a group administrator will define several attributes such as the name, what resource templates can be used, and when the group expires. They will also set the default budget allocation for each member of the group. When a user is added to the group, they will be given that budget amount by default. Altering the group’s default budget amount will not change the budget allocation of any users already assigned to the group. The group’s admin/managers will be able to alter the budget quotas for individual users at any time. 

> **Backlog/JumpIn:** SCAMP v1.0 will not track if the total budget allocated to all users exceeds the budget allocation for the group. This would need to be checked as users are added to the group or user budget allocations are changed. **[Linked to GitHub Issue #152](https://github.com/SimpleCloudManagerProject/SCAMP/issues/152)** 

Please note that if a group budget allocation is reduced (or the group removed), only that portion of the group’s allocation that has not yet been used will be returned to the group admin for reallocation. This will prevent a group admin from re-using budget allocations by removing and recreating groups.

## Units, Templates, and Currency Rates ##
Measuring consumption in Azure is a bit complicated. Different resources are measured in different units of measurement and on different time frames. Azure Database (SQL) is billed by the day, DocumentDB collections are by hour, and Azure Compute (VMs) by the minute. Add into this mix that we want SCAMP to eventually support globalization by allowing the System Admin to set the currency that should be used by the system and well… it’s complicated.

In looking at this situation, we decided that SCAMP itself would track resource consumption in terms of an arbitrary unit of measure that we’ll refer to as “**consumption units**”. Each resource deployed via SCAMP will be based on a SCAMP resource template (more on that in another document). Among many attributes, the template will define the resource’s “**burn rate**”. This is the number of units the resource will consume over the **minimum billable time** (minute, hour, day, etc…) for that resource. 

This concept of “units” is translated into currency at the SCAMP system level. When SCAMP is set up, the system admin will define what currency type (Dollars, Euros, etc..) and how much of it each “unit” is worth. This is then used in all displays of consumed and remaining budget amounts. 

Over time, as resource pricing changes and currency exchanges fluctuate, the system admin will be able to adjust the burn rate of resources and the current value of units to compensate. But ultimately, our hope that we’ll be able to leverage resource tagging and eventually an Azure provided billing API to measure usage. Until then this setup should work nicely. 

The downside is that we’ll need to track budget allocations and the consumption against them at the group admin, the group, user within a group level. Each has their own role to play and budget to manage against.

## Azure Web Apps vs Virtual Machines ##
Another aspect of consumption in Azure is if you are billed for individual resources, actions against a resource, a “container” in which resources exist. Examples of individual resources are items like DocumentDB collections and IaaS virtual machines. Operation based billing is items like Azure Storage and Service Bus Queues. Azure Web Apps is an example of the “container” based model.

> Note: SCAMP does not currently plan to provide transaction based resource monitoring. Our expectation is that the designer of a SCAMP resource template will adjust the burn rate of the resource accordingly to compensate for factors that can’t be easily monitored at a resource level today. This includes items such as the bandwidth and storage transactions consumed by a running virtual machine. We believe this method can provide a “good enough” estimation. This decision will be reevaluated based on the needs of solution adopters.

Unlike IaaS Virtual Machines, Azure Web apps are associated with a hosting plan. The charges are against the plan. This means that for SCAMP, we need to track the plan (to measure usage against), while also keep track of the web sites within the plan. 

Our plan is to handle this to have one resource for the hosting plan. This resource would be automatically assigned to the group admin that created the group and visible to them in the SCAMP portal and it would have the appropriate burn rate as defined by the template it was created from. As users of the group create additional resources based on this template, they would be associated with the “hosting plan” resource, and have a zero burn rate. This parallels the way we believe group administrators will also be handling “shared” resources.

To go back to our earlier scenario of an instructor teaching a class on digital content publishing. The instructor would provision the web hosting plan as well as possibly a VM running MySQL for the students to attach their web sites too. The group administrator would be able to view/manage these shared resources since they “own” them, but the students would only be able to manage their individual web sites.  

## Resource Monitoring ##
Because SCAMP will exist outside of Azure, and in the cloud “stuff happens”, resource monitoring is not just a case of looking at SCAMPs activity logs. SCAMP will use an autonomous process that performs two key tasks:

- Reconcile the state (running, stopped, deleted, etc) reported by SCAMP of any resources its currently managing against their state in Azure. 
- Increment the amount of budget consumed by the resource for the owning user, the group the resource is in, and the group manager that created the group. 

The process will be implemented as an ASP.NET vNext console app that can be run continuously as an Azure Web Job. This application will be the **SCAMP Resource Monitor**.

The workflow for the monitor will be as follows:

- Query the SCAMP database for the list of all SCAMP resources
- For each resource, look up its current state in Azure
	- If the SCAMP state of the resource differs from Azure’s reported state, update the state
	- If the resource is in a billable state, update budget consumption

The tricky part here is the update of the consumption metrics (how much the resource has “cost”). Fortunately, my colleague [Gabriele Castellani](http://www.gabrielecastellani.it/) from Italy had already put some excellent thinking into this and [created a slide show](/docs/BudgetManagement.pptx) that outlined nearly all of what follows. 

As previously mentioned a budget itself has the following attributes: start & end dates; billing period (on end or # of days); total & available budget. These are set by a system admin when they assign group admin permissions. The group admin then allocates a portion of that budget out to each group they create/own, which is in turn allocated out to each user. As the users create resources, these will consume the allocated budgets at each level.

When a resource enters a consuming state (is started/created), it will be given a consolidation time for use in consumption calculations that is the time it entered this state plus the resource’s minimum billable time. It will also update the upstream budgets (user, group, and group admin) with the initial minimum usage (minimum billable time * units * burn rate).
 
As the Resource Monitor performs its regularly scheduled check of all resources, it will check to see if the resource’s “consolidation time” has passed. If it has, it will update the upstream budgets as described above. It will also update the resource, advancing it to the next available consolidation time. If the consolidation period spans budget periods, the system will take this into account and only apply that usage which applies to the current billing period. 

The update of consumption will also be done each time a resource enters a non-consuming state. This way if a resource enters and exits a billable state multiple times between scheduled runs of the Resource Monitor, we can track that usage and not let if fall through the cracks and go un-recorded. 
Here’s an example…

> We create a virtual machine that has a consumption rate of 40 units per 60 minutes (the burn rate) and a minimum billing period of one minute. The VM is created (enters a consuming state) and we update its next consolidation time to “now + 1 minute”. We also update all the upstream budget levels: (40 units / 60 minutes) * 1 minute of usage = 0.66 units consumed.
>  
> 5 minutes later, the Resource Monitor sees the VM is still running and that the consolidation period was 4 minutes ago. It resets the consolidation period to “now + 1 minute”, and updates the upstream budget levels with: (40 / 60) * 4 = 2.64 units consumed. 
> 
> A similar calculation is made 3 minutes later (2 minutes after the last consolidation) when the VM is stopped: (40 / 60) * 2 = 1.32 units consumed. 
> 
> The general formula here being: burn rate * duration = units consumed

Similarly, if the resource has a longer minimum billing period, say 1 day… 

> We create an Azure SQL Database that has a consumption rate of 2 units per hour and a minimum billing period of 24 hours. The resource is created and we update its next consolidation time to “now + 1 day”. We also update all upstream budget levels: 2 (burn rate) * 24 (minimum billing period) = 48 units consumed. 
> 
> 5 minutes later, the Resource Monitor is still consuming but that the consolidation period hasn’t yet expired. 
> 
> 3 minutes later the database is removed. Since we still haven’t passed the consolidation period, the upstream budget levels are not updated as we’ve already applied the minimum billing period for this resource.
>  
> If the database was NOT removed, Resource Monitor would continue to look at the resource until its consolidation period expires. It would then calculate usage (nn minutes with a minimum billing period of 1 day, so 1 day worth of usage), subtract that usage from the upstream budgets levels, and reset the resource to the next consolidation period (advance by 1 day). 

As the Resource Monitor does the updates at associated levels, it will also see if it’s time to “reset” the amount consumed for the budget(s). Scamp will track when the billing period for a budget expires, and when that target passes, it will zero out the used budget at all levels. Starting all resources at “zero” for the new billing period. 


> **Backlog:** SCAMP may eventually store past billing periods so that users will be able to see historical usage trends over time. 

> **Note:** This design does not currently support “role-over” of usage. The system admin will need to properly define templates with a small enough minimum billing period to support any needs they may have. 

If the available budget at any of these levels reaches zero, the system will then take the appropriate action to eliminate/reduce additional consumption based charges. Those corrective actions will in turn generate managed events (sent via an Azure Storage Queue) that will be processed by the Resource Manager so they can be applied to Azure. 

Each resource template will need to define what action(s) should be taken when the budget is exhausted. A VM can be placed into a stopped/deallocated state in which only storage charges continue to occur. But a web site plan can only be scaled down (problematic) or deleted to stop incurring charges). This will be part of the SCAMP resource templates so that each deployment of SCAMP can make their own decisions about how to handle this.
 
## It’s a start… ##
It’s not a perfect design, but the hope is that this will get SCAMP by until such time as there’s an API available in Azure that can be leveraged. When this effort first stated, we thought it was going to dive deep into measure usage metrics on various resources (bandwidth used per web site, storage transactions per VM disk)… and while our early research found we could capture many of these, there were still many gaps. Ultimately, we realized many of those items would require significant effort for minimal return and by properly estimating the burn rate of resources, we felt we could get resource monitoring to a point that was 90-95% accurate. This level of accuracy was “good enough” for the customers we talked with. So we’ve proceeded down this path of resource monitoring. 

