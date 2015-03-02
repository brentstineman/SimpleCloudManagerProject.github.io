---
layout: article
title: "Getting Started"
date: 2015-03-02 07:36:00 -8000
author: brent_stineman
---

Here we go, the inaugural blog post of the Simple Cloud Manager Project. We've been building this site out for the last couple weeks and while I'm still not satisfied with it, there reaches a point where you just have to say its time to move on to the next task. And those tasks are about starting to write some code.

##Choice of Technology##
So the first question I've been getting lately is why did we select the technologies that we did. Honestly, it was a mix of personal preferences (what architecture isn't driven at least in part by personal preferences), and in part by a desire to learn. 

At heart, I'm a problem solver. And its fairly easy to get caught up in just solving the problem with the tools you're most familiar with. And being mostly a C# developer for the last 10 years, it would have been very easy for me to just pick that tool and run with it. But as I looked around as some of the brilliant members of the team I'm part of, I'm keenly aware that my focus on C# has blinded me to much of what's been going on in the world of development. So for this project, I wanted to embrace some of these new challenges. 

To this end, we quickly set our eyes on things: the user interface, which we knew needed to be a web site, needed to leverage the quickly growing world of Single Page Apps (SPA). Angular is the current front-running, but as I talked with Felix I realized that Ember is something our team has invested fairly heavily in. And in some ways, the world of Ember more closely aligned with the MVVM style experiences I'd already had. Ember is quickly growing in adoption, and given the experience of our wider team it meant I'd have some good folks to lean on as we learn this new approach. 

The second item I wanted to leverage was for the back-end of the project. We'd already explored what the new Azure Resource Manager model could provide and knew it was the plan for Azure management going forward. We could code directly against its API, but felt that given our skills in C#, there was another learning opportunity here that we could leverage. Namely, building the WEB API to be used by the Ember site with ASP.NET 5. This allowed us to learn the next generation of ASP.NET, while also leveraging our existing skills to easily interact with the .NET based Microsoft Azure Management Libraries (MAML) v2.

Together, these selections also provided a 3rd opportunity. To have a functioning project that combines the open source world (Ember/Javascript), with the Microsoft world (ASP.NET/C#). This would highlight that the two can co-exist and give us a way to learn first hand what works, and what could use improvement. Learnings we can then share with our customers, as well as with the groups that produce this great project. 

And through all this, the team would grow. And that growth, the learnings, those to me can be some of the most significant benefit of any project. And to a significant extent, is what my team... Strategic Engagements is all about. 

##User Stories##
But enough about tech, lets get back to the goal at hand. Building the Simple Cloud Manager. First, we need to define some user stories based on our requirements so we can lay our some tasks and get to work. 

The project has been fairly quiet for the last two months and all that changes today. Last week the core team and myself met and we discussed three stories that will be at the heart of the first preview release we want to make available in early April.

#####Group Management/Setup#####
Groups are a central concept for SCAMP in that all resources are placed into groups. These groups then have their own administrators/owners and optionally usage quotas. Groups will also eventually control various defaults on the resources that can be used. For example, one group may only be allowed to use a specific virtual machine image, or may be restricted to just web sites. 

[Shawn Cicoria](https://twitter.com/cicorias "Shawn Cicoria @ Twitter") has taken on the job of detailing this story and the feature list that will comprise it. 

#####Group Administration#####
Once a group is created, the owner will set about assigning users to the group and allocating resources. This falls into the next story being handled by [Brad Merrill](https://github.com/zbrad "Brad Merrill @ Github"). 

In this story, we start to see one of the key aspects of SCAMP come to life. Imagine a group as a class with students, a dev camp with attendees, or even a project with team members. Inside this group go the resources that will be used by the group members. 

#####User Resource Interaction#####
This all of course comes down to users being able to access their resources. Once user has been made a member of a group, and a resource allocated to them, they can then take actions against the resource like provisioning it, starting and stopping, or deleting (aka de-provisioning) them.

This is also where we will see the beginnings of the usage tracking come into play. Each action will be logged and if time allows, we'll see if we can't sneak in the "automatic shutdown" feature. 

[David Makogon](https://twitter.com/dmakogon "David Makogon @ Twitter") has volunteered to take this story and help drive it out. 

##Going Forward##
So as I look back over the last few months, there's been a lot of "duck on the water" syndrome. For those of you not familiar with this analogy, it refers to a duck swimming on a still pond. Above the water, you see a duck just moving slowly along. Not much appears to be happening. But beneath the water the ducks feet are moving rapidly. 

That's kind of what's been going on with this project. Over the last 2 months I've been working to get all the approvals required for a company like Microsoft to engage in an open source project like this. But we're nearing the end. I'm waiting for our final approval which will hopefully come this week. I'm also nearly done with another internal project which means that from now through the end of Microsoft fiscal year, this project will represent the majority of my time. 

I've also gotten a commitment from most of the team that the week of March 16th will represent out next "hackfest". A few of us will be getting together at the Microsoft Campus in Redmond, Washington for the purpose of helping leapfrog some of the stories I mentioned above. We also have a few new contributors that will be joining us from around the globe to help with the effort going forward. 

As if this wasn't enough, you're also going to start seeing us putting more issues up on Github. ANYONE can comment on the issues, so if you have a vested interest in the project, look for issues "[Discussion](https://github.com/SimpleCloudManagerProject/Scamp-Web-Client/issues?q=label%3ADiscussion)" or "[Feature](https://github.com/SimpleCloudManagerProject/Scamp-Web-Client/issues?q=label%3AFeature)".

We should also hopefully be getting details squared around in the next few weeks with regards to external contributions. That's been waiting on the final internal approvals as well as us getting a fully defined task list out. 

So in closing, thanks for paying attention to our little project. And I hope you keep coming back to see the latest. The next few months should see some hopefully exciting work taking place. 