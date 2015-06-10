---
layout: document
categories: ["docs", "database"]
permalink: /docs/datadesign/
title: "Data Storage"
collection: "docs"
category: documentation
share: false
description: Background and explanation of SCAMPs approach to data storage
---

Every project needs a feature list. So as we begin the Simple Cloud Manager Project, we drafted a list of features based on the requirements of potential early adopters, namely educational institutions and small/medium sized businesses.

## Provide a web based "portal" to interact with the system
The primary method of interacting with SCAMP will be via a **web site** that must work properly across the majority of the browsers and devices used by the initial target audience.

The web portal must be able to be "**white labelled**", so that the organization leveraging it can make the site match existing web assets.  Security for the portal will initially be provided via **integration with an existing Office 365 or Azure Active Directory domain** (this will hopefully be expanded in later versions). **Users will not be required to also be on the Azure subscription**. 

And the portal must be **easy to use**. If end users need to be "trained" to use it, its not simple enough. 

## Allow users to access and provision resources
We envision 3 primary user roles:

- System Administrators: responsible for SCAMP configuration and subscription level administration

- Group Administrators: manage a group of resources and the users that can access them.

- Users: consume resources assigned/allocated to them by a group administrator

SCAMP will also offer a degree of **control over the types and location of resources**. For example, an Administrator could **restrict which Azure regions can be used**, and which **Virtual Machine gallery images and sizes are available**. SCAMP will **provision and distribute resources across the available Azure subscriptions** as needed. 

In V1, we plan to allow you to use SCAMP to **manage Azure Web Sites and Virtual Machines** which will in turn be **organized into groups**. Users will only be able to access the resources assigned to them, or that are part of a group that they are the administrator of.

One of the trickiest features we hope to provide in V1, is the ability to **push button creation of a gallery image from an existing virtual machine**. Simplifying this task sufficiently so that a non-technical user can create a reusable image that can be shared with other users.

## Track and Control Azure costs
First and foremost, we must put in place a way to track the billable usage of users and place some controls on them.

**Usage quotas** can be placed on groups, or individual resources. We adopted the term "quotas" because we cannot currently provide the level of billing measure that would be needed to enforce a fixed cap. So SCAMP will **capture usage metrics** that are available, and provide a way to **estimate usage**. 

The system will **track all user actions**, allowing for simple auditing and state tracking. This will allow automation for cost control such as **automatically shutting down virtual machines** after a given amount of time or when usage quotas are reached.

In addition to setting quotas, the system will even allow for **setting of your own rates** and the **billing period** to be used and if that period is recurring or non-recurring. 

And by **tracking both current and historical usage**, SCAMP will allow you be able to **view and export usage details** to create reports.
