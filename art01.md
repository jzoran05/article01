
Keywords: governance, platform, service, azure, application, teams, shared

# Outline
- 
- scope enterprise organizations (medium and large)
- Approach: many options, this article is describing one of options (may not be ideal) but it is tested (?)
- Application vs. platform (service)
    - Application Service owns multiple components hosted within platform service. Example APi deployed within API Management platform. API belongs to Application (Microservices principles). Figure: Application boundary with multiple components (e.g. Messgig Topic, API, etc)
- Shared infrastructure / platform services (you can call it differently)
    - platform layers (datacentre, hardware, virtualization, ...)
    - not necessarely all platform services have to be shared/centralized. Cloud allows easy provisioning and management without cost overhead (but maybe operational overhead). But thi sarticle focuses on scenario where platform service is shared.
- API, messaging topics, reports are all part of a Microservice - these are Microservice interfaces 
- Separate lifecycle (and teams) for platform and application service
    - requires separate CD pipelines
    - updating, testing the platform
    - API exposed by an Application should be part of App lifecycle (API is not a separate Service - unless API is composition of multiple APIs but in this case this is not just an API - it is an Service which also exposes an API)
- Multiple approaches
    - No any centralization (each application has instance of platform service 
        - may not work in case of platforms for which pricing model is not cost effective for single application
        - may not work in cases platform is also used as system of record (information still can be retrieved by requires some kind of report to be built in order to retrieve and centralize the information e.g. list of APIs
        - Describe deployments and lifecycle in this case (?)
            - Common release for multiple applications - application teams wait for release schedule (schedule may be on weekly basis?)
    - Platform services are centralized/shared
- single Application/Microservices = single Resource Group
- Deployment of Application services within/over shared platform services 
    - by leveraging ARM
    - each application team owns their application deployment (but deploys to platform owned by shared platform team
    - ARM platform deployments with separate referenced ARM files)
- Demos
    - Service Bus (as shared platform)
    - API Management (as shared platform)


# Summary (?)


# Introduction
  
In many enterprise organizations IT departments typicaly organized themselves by having multiple application development teams and by centralizing Infrastructure team. These teams may be called differently in different organizations like Operations team, ... These teams are mainly focused on designing and deploying infrastructure components and operating both infrastructure and applications hosted by the infrastructure. 
The larger the organization, the more separations exist within such team as well and in these cases these could be several groups each with specific more granular responsibilities e.g. separate group for application operations and separate for infrastructure operations.
Typical information system for such organization consist of over hundred of applications. Infrastructure and operating cost is usually not insignificant and organizations are typicaly measuring Total Cost of Ownership (TCO) per application and trying to reduce it where possible. This often leads to shared infrastructure platforms. Usually such concepts are tradeof between increase the density of Applications per Infrastructure servcice which reduces the cost but on the other hand centralization of such infrastructure services is usually impacting Applicatioin lifecycle independance.
Cost is not the only reason for such shared or centralized infrastructure platforms. Another reason is importance of centralizing the information about deployed components or artefacts. 

This article describes one of possible ways how both Application and Infrastructure teams could organize and manage ownership of Microsoft Azure services (resources) used by both Application services and centralized or shared Infrastructure services. 
How to decide which infrastructure services should be shared is out of scope from this article.


# Shared Infrastructure or Platform Services

I'll try to introduce concept of shared infrastructure or platform services and why and when it makes sense - please skip this section if you feel that there is nothing new you need to read about this topic.
In order to accelerate software delivery, development teams have tendency to self-contain their Application Services and minimize number of external dependencies. "External" in this case may be dependency to external team as well. No matter are you traditional waterfall project team or an Agile team, for each external dependency your project team has to align planining, interfaces, SLA, etc with external team and this takes time. Use of shared infrastructure / platform service is no different.
But on the other hand, in many cases this is either mandated by some organization body or it is simply inpractical to do it otherwise.

Number of years ago I have witnesed reluctance of application development teams to use shared services platforms. In this concrete case the shared platform was a shared web farm. It consisted of larger number of physical machines (back then) with windows operating system and internet Information Server (IIS). Idea was to ofer to each team Web Site within IIS. Apart from quite buerocratic approach to request onboarding to the platform, applications also often experienced instabilities which were often caused by CPU or memory overutilization by one of bad bahaving applications which, becasue there was no any resource isolation, was impacting other hosted web applications. Apart from these dificulties, shared web platform had scheduled platform maintenance (which were frequent which meant application downtime) and scheduled deployment time slots (which were quite infrequent) and applications teams felt uncofortable using the platform. Many application teams decided to trade of a lower cost for independance and purchased their own hardware and created application dedicated infrastructure environments - which were provided and operated also by Infrastructure team but they didn't have the same constrains as for shared platform.
Today we have much more advanced infrastructure services and resurce isolation is not that of the problem any more but other constrainst I have mentioned in the example are still applicable and if from cost and operational perspective this is acceptable, an application dedicated infrastructure or platform environment is certainly not a bad choice in many cases. In a lot of cases, dedicated infrastructure or platform per application is simply inpractical or very cost ineficient. Let me provide couple of examples.

It would be very inpractical to deploy separate API Management platform for each application. When I say platform, I mean the whole typical API Managment platform which consist of several components like Publisher and Developer portals, API Gateway cluster(s), API analytics (which could be integrated with both Publisher and Developer portals). In the public cloud environments, it is very easy to provision new API Management platform instance. Even from cost perpective you may chose a tier which is a cost efficient for single application but it would be very problematic from usability perspective for Application teams. Finding and subscribing to an API in this case would be a quite difficult. The most likely all teams will have to publish another centralized tool to record information about their APIs, where are they located (URL) which is actually one of purposes of API Management platform. Gathering cross application API analytics would be impossible (without additional centralized reporting) and so on.
Now an organization doesn't necessary need to have only single API Management platform instance and it may chose for many reasons to deploy multiple for either segmenting by APIs usage type (Public, Private, etc) or segmenting per line of business (Consumer Banking APIs probably don't need to be hosted in the same API Management platform instance for Corporate clients banking APIs).

Another maybe a little bit less obvious example compared to API Management platform could be a Messging plaform. A preatty much everythign said for API Management platform is applicable to Messaging platform with a difference that most (if not all) of the Messaging platform are lacking strong API catalog like organizing and searching feaures but they are not too far from API Management platform in that sense.

Now maybe an exammples where shared infrastructure or platform service is not a must.
If I have a aPaaS service and in this case I am not aware of underlying (virtual) servers maybe I don't need anymore to optimise the cost by hosting multiple application on the same set of (virtual) servers. Of course, who ever provides aPaaS service still deals with servers and manages these resources for the consumer but that is not the customer's problem any more - customer pays for what it uses.
Also, Operations wise, it is very debatable if there is any operational efficiency by hosting multiple applications within single aPaaS service.
Let's use Azure App Service for an example. App Service has to use App Service Plan. Now if you make a parallel with pre PaaS era, you can say that App Service Plan is the same as Web Server farm and App Service is just an Web Site (which is actually not far from the truth). There is very litle or no resource isolation between App Services hosted within the same App Service Plan. App Service plan has a "farm" of servers - virtual machine scale set. Scaling-out or up of the farm afects all App Services hosted within the App Service Plan.
There is very little Operations team has to really do on a regular bases in order to operate and maintain the App Service and App Service Plan. Apart form monitoring and having alerting configured most of not all of changes required to be performed within App Service is requested by Application team. Changes that are required across all of the App Services whcih may be originated by Operations team should be automated anyway - so applying the change on single App Service or on all App Services deployed by the organization is not (or it should not be) any more time wise expensive.
So I would argue that cost efficiency in case of aPaaS should be a reason to use aPaaS as a shared platform.

# Line between Infrastructure or Platform service and Application Service


# References 
    - (governance book about services, etc)
    - Azure ARM template documents