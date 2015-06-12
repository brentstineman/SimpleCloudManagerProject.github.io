---
layout: document
categories: ["docs", "database"]
permalink: /docs/features/datadesign/
title: "Data Storage"
collection: "docs"
category: documentation
share: false
description: Background and explanation of SCAMPs approach to data storage
---


> Please use [this thread](https://github.com/SimpleCloudManagerProject/SCAMP/issues/151) for any feedback or questions regarding SCAMP's approach. 

SCAMP, the Simple Cloud Manager Project, is a tool to remove complexity from cloud resource management (such as VM allocation), for well-defined use cases (such as a classroom training environment). Varying resources may be created and managed, along with specific resource constraints (e.g. VM size, maximum clock-time, valid from-to dates, etc.). 

SCAMP requires a database to track human resources, cloud resources, permissions, and consumption for a given subscription. Given this, coupled with the fact that SCAMP is published as an open source project, the database selection needed to fit specific guidelines: 

- Simple to access. No server-side database installation required. 
- Freely-available language libraries 
- Scalable to meet the projected needs of a school deployment such as Georgia State University 
- Capable of working with the flexible schema required by SCAMP 
- Document vs Relational Store

Given the requirement for a service-based database solution, this narrowed the choice: 

- Azure SQL Database (a relational store, providing a rich subset of SQL Server) 
- Azure DocumentDB (a document-store database, similar to MongoDB) 
- Azure Table Storage (a key/value store) 

The nature of SCAMP is that most of its data will be read far more than it is updated. This immediately brought to mind the CQRS storage pattern. Among the many aspects of this pattern is the idea that “views” are rendered based on how data needs to be presented and that updates occur out of band. This pattern is based on eventually consistency models that are intended to allow systems to scale incredibly well. Alternatively, this pattern also makes the rendering of views very cost affective. 

RDBMS have been around for some time so Azure Database represents a familiar model. However, scaling this type of a database can present some challenges even with the features offered by Azure Database. But the patterns around it are well established and there’s no shortage of folks (especially among the SCAMP team) with SQL skills. 

- Ultimately, we selected Azure DocumentDB for several reasons
- It could easily scale to the upper bounds of our needs (10,000 Azure resources)
- It would help keep the load on the database and on the application light
- No fixed schema to maintain, so enhancements should be easier to make
- Alternative database systems are “in” and using it would be a good exercise for the team

The last item has little to do with the technical implementation. But if you’re going to undertake a project, it’s often times helpful to sprinkle in something you want to work with if it fits. The team consensus was that DocumentDB would be a good fit.
 
## DocumentDB challenges  ##
There are still a few features absent from DocumentDB, given the newness of the service, which are provided in alternative document stores such as MongoDB: 

- Ability to backup/restore data (today, this requires a custom backup solution with DocumentDB, as there are no facilities to snapshot / backup a collection) 
- In-place updates (e.g. REST PATCH verb). This is a committed feature but is not yet available 
- Scale-out. With a 10GB size limit per partition, sharding becomes necessary. Fortunately the .NET SDK provides this now. Additional SDKs will eventually see this as well. 
- Wire protocol. Today, only the .NET SDK provides direct TCP communication between client and DocumentDB, while the other language SDKs are http-based, via a gateway. The proprietary protocol has not been open-sourced, so there’s no way to currently implement this in other SDKs yet.
 
However, the startup cost of, say, MongoDB (requiring minimum 3 VMs), along with related monitoring and maintenance, gives DocumentDB a clear advantage, especially for SCAMP (which has no need for MongoDB-specific features such as the Aggregation Framework). 

## The Design ##
SCAMP database design centers around three core entities:

**ScampResource** – The attributes that describe an Azure resource (which may or may not currently exist). 

**ScampUser** – a representation of the user dashboard view. It contains summary information for all the groups the user is a member of as well as the resources they own/control. 

**ScampResourceGroup** – The logical grouping of users and resources for administration/management. Contains summary information about the users and resources that are part of the group.
 
Each of these entities/documents represents a way of looking at the information. ScampResource contains references back to the resource owner(s). A summary of the ScampUser info (Id and Name only) for those related entities. ScampUser likewise has summaries of the resources as well as the groups those resources are a part of. Finally, ScampResourceGroup has summaries of both the users and resources that are part of the group. 

This redundancy flies right in the face of all we’ve been taught about RDBMS normalization. This is not the only way to think of using document based data stores. For SCAMP, we based our design off the ways the data is most often be accessed. You will notice that the resource does not contain a reference back to its owning group. This is because when we walked through the design for the system, we realized we would not really need to start at the resource level and climb “back up” the tree to its owning group, so there was no need to include the linking information in the resource view.

## Learning a new database ##
SCAMP, above solving a couple key problems, is also about learning and demonstrating those learnings. With DocumentDB, the team had some learning to do. 

First off, we had to be conscious of cost concerns. DocumentDB charges based on [performance levels (a request unit capacity)](http://azure.microsoft.com/en-us/documentation/articles/documentdb-performance-levels/) which are assigned to collections. For cost considerations, we opted not keep each document in a separate collection and use a type attribute to designate the different documents. 

This was a design decision and not based on any feature limitations of DocumentDB. We only had a limited number of document types and felt that the majority of SCAMP implementations could get by on a single, or only a couple of Performance level units. 

We also could have gotten by without using a “type” attribute. As was pointed out to me by a good friend and Document DB expert, if I had an entity called pet and another called child, instead of creating child with name and pet with name I would create child with childName and pet with petName. That would let me do SELECT * FROM c WHERE petName = ‘fluffy’ and ensure I only ever got back pet documents.

The next lesson we learned was that we couldn’t search a document for partial matches within a given attribute/field/property.. For example, looking up a user by a partial name. However, we can easily combine Azure’s Document Database with its Search service. Azure Search also had cost implications based on the number of documents to be indexed. But we found a way to limit which documents Azure Search would index… 

When creating the Azure Search datasource, there is an optional “query” attribute. Here’s an example of the payload used in the create datasource API call… 

	{ 
	    "name" : "scampdocdb",
	    "type" : "documentdb",
	    "credentials" : { "connectionString" : "AccountEndpoint=https://<ourDBURL>;AccountKey=<accountkey>;Database=<databasename" },
	    "container" : { 
	        "name" : "scamp1",
	        "query" : "SELECT * FROM docs where docs.type = 'user'"
	    }
	}

For SCAMP we determined that there is no need for a Search index. We were going to do partial matches based on name but initial requirements are for exact lookups based on Azure AD user Id. So we’ve filed that research away for another day in case those needs change. Next we found there was no way to do cross document joins. In hindsight, this seems well… obvious. The issue was that as a team, we were still thinking in RDBMS terms and hadn’t fully embraced the notion of a document representing a “view”. Therefore, instead of doing some type of join query between a “user” and a “resource” document, we instead opted render the “user view” as the user document. Something like… 

	{
	  "id": "a144e3bd-5b25-4162-b927-3c17e4f7235a-424b3202-cc65-4f93-88e5-38a19b9116b2",
	  "name": "Brent Stineman",
	  "type": "user",
	  "groupmbrship" : [
	    {
	    "id": "3444789d-e889-4127-b6c2-baeb59c39d09",
	    "name": "GSU Hackfest",
	    "isAdmin": true,
	    "resources": [
	      {
	        "id": "8807789d-e889-4127-b6c2-baeb59c39d09",
	        "name": "BrentStineman-GSUHackFest",
	        "type": 1
	      }
	    ] 
	    }
	  ]
	}

There is still one lesson we had to learn. With this new “view” approach, we have the need to have to update several documents in a way that provides a degree of transaction control. Document DB provides this type of functionality via its “Stored Procedures” These are defined in JavaScript that we can then execute via .NET (our API). 

The result is that we are confident we have an approach that will scale well and meets our needs for both function and cost. But document databases are decidedly a far cry from the traditional RDBMS approach most of us have used over the last few years. This difference however, makes it no less viable approach. 

## Volatile Data ##
Last week during the hackfest, one of our new participants, Khaled, noticed something that had been overlooked. While the majority of the document data is fairly static. This is the basis for our CQRS style “view” design. The  remaining data is more volatile because of our need to track resource usage. Compound the fact that our schema would have resulted in something of these volatile elements being stored in multiple views as part of the CQRS model… and our attempt to minimize DocumentDB resource consumption quickly falls apart. 

The team discussed this with Kal and he proposed we use Azure Table Storage for this type of information. We’d already planned to use it for activity logging. So it wasn’t a far cry to adopt it for the other volatile resources. The approach would use a single table to store the state and usage/consumption details for every resource being managed by SCAMP. A standard Azure Storage account can scale up to 20,000 operations per second as long as it is properly partitioned.

We wanted to leverage the performance units in DocumentDB in a way that balances cost and complexity.  If we kept the usage metric in our documents, we need to update the docs for any active resources each time we updated usage metric tracking. This could be once every five minutes. Each of these updates would require changes to three different document views, and we need to take into account there will be a risk of update collision (two processes trying to update the same document at the same time) on at least two of the three primary SCAMP view documents as many resources share them. 

If you use SCAMP to manage a significant number of resources, our cost concerns for DocumentDB would evaporate. At the lower end, we felt that putting the volatile elements (there is only two of them) into ATS gave us a better cost/complexity ratio awhile also giving the team as a whole a much better feel on the predictable scalability. 

Admittedly, this is only easily scalable up to 20k operations per second (the limit of a single Azure Storage Account), but at even 10k resources updated every 5 minutes, we shouldn’t end up hitting that upper limit very easily. If/when DocumentDB gains a “patch” feature, this may make it worthwhile to re-evaluate the decision. Meanwhile the “upsert” capability of a tiny Table Storage entity should work really well.

Sprinkle in a little caching, the result will be highly scalable and performant for SCAMP. 

## Polyglot Persistence for the Win! ##
So there you have it. Not my best write-up, and certainly an architecture that could be debated over numerous adult beverages in the gin-joint of your choosing, but architecture is often “more art than science”. 

We are confident we have something we are confident will perform well for our v1.0 release and hopefully will be easily maintained by those that decide to use and (hopefully) contribute to SCAMP. 






