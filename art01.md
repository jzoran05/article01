# Azure Shared Platform and Application Service lifecycle management (draft)

**Keywords**: *governance, platform, service, azure, application, teams, shared*


# Outline

- scope enterprise organizations (medium and large)
- Approach: many options, this article is describing one of options (may not be ideal) but it is tested (?)
- Application vs. platform (service)
    - Application Service owns multiple components hosted within platform service. Example API deployed within API Management platform. API belongs to Application (Microservices principles). Figure: Application boundary with multiple components (e.g. Messaging Topic, API, etc)
- Shared infrastructure / platform services (you can call it differently)
    - platform layers (datacentre, hardware, virtualization, ...)
    - not necessarily all platform services have to be shared/centralized. Cloud allows easy provisioning and management without cost overhead (but maybe operational overhead). But this article focuses on scenario where platform service is shared.
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

# Introduction
  
In many enterprise organizations IT departments typically organized themselves by having multiple application development teams and by centralizing Infrastructure team. These teams may be called differently in different organizations like Operations team, ... These teams are mainly focused on designing and deploying infrastructure components and operating both infrastructure and applications hosted by the infrastructure. 
The larger the organization, the more separations exist within such team as well and in these cases these could be several groups each with specific more granular responsibilities e.g. separate group for application operations and separate for infrastructure operations.
Typical information system for such organization consist of over hundred of applications. Infrastructure and operating cost is usually not insignificant and organizations are typically measuring Total Cost of Ownership (TCO) per application and trying to reduce it where possible. This often leads to shared infrastructure platforms. Usually such concepts are trade-off between increase the density of Applications per Infrastructure service which reduces the cost but on the other hand centralization of such infrastructure services is usually impacting Application lifecycle independence.
Cost is not the only reason for such shared or centralized infrastructure platforms. Another reason is importance of centralizing the information about deployed components or artefacts. 

This article describes one of possible ways how both Application and Infrastructure teams could organize and manage ownership of Microsoft Azure services (resources) used by both Application services and centralized or shared Infrastructure services.

# Shared Infrastructure or Platform Services

I'll try to introduce concept of shared infrastructure or platform services and why and when it makes sense - please skip this section if you feel that there is nothing new you need to read about this topic.

In order to accelerate software delivery, development teams have tendency to self-contain their Application Services and minimize number of external dependencies. "External" in this case may be dependency to external team as well. No matter are you traditional waterfall project team or an Agile team, for each external dependency your project team has to align planning, interfaces, SLA, etc with external team and this takes time. Use of shared infrastructure / platform service is no different.
But on the other hand, in many cases this is either mandated by some organization body or it is simply impractical to do it otherwise.

Number of years ago I have witnessed reluctance of application development teams to use shared services platforms. In this concrete case the shared platform was a shared web farm. It consisted of larger number of physical machines (back then) with windows operating system and internet Information Server (IIS). Idea was to offer to each team Web Site within IIS. Apart from quite bureaucratic approach to request onboarding to the platform, applications also often experienced instabilities which were often caused by CPU or memory overutilization by one of bad behaving applications which, because there was no any resource isolation, was impacting other hosted web applications. Apart from these difficulties, shared web platform had scheduled platform maintenance (which were frequent which meant application downtime) and scheduled deployment time slots (which were quite infrequent) and applications teams felt uncomfortable using the platform. Many application teams decided to trade of a lower cost for independence and purchased their own hardware and created application dedicated infrastructure environments - which were provided and operated also by Infrastructure team but they didn't have the same constrains as for shared platform.
Today we have much more advanced infrastructure services and resource isolation is not that of the problem any more but other constraints I have mentioned in the example are still applicable and if from cost and operational perspective this is acceptable, an application dedicated infrastructure or platform environment is certainly not a bad choice in many cases. In a lot of cases, dedicated infrastructure or platform per application is simply impractical or very cost inefficient. Let me provide couple of examples.

It would be very impractical to deploy separate API Management platform for each application. When I say platform, I mean the whole typical API Management platform which consist of several components like Publisher and Developer portals, API Gateway cluster(s), API analytics (which could be integrated with both Publisher and Developer portals). In the public cloud environments, it is very easy to provision new API Management platform instance. Even from cost perspective you may chose a tier which is a cost efficient for single application but it would be very problematic from usability perspective for Application teams. Finding and subscribing to an API in this case would be a quite difficult. The most likely all teams will have to publish another centralized tool to record information about their APIs, where are they located (URL) which is actually one of purposes of API Management platform. Gathering cross application API analytics would be impossible (without additional centralized reporting) and so on.
Now an organization doesn't necessary need to have only single API Management platform instance and it may chose for many reasons to deploy multiple for either segmenting by APIs usage type (Public, Private, etc) or segmenting per line of business (Consumer Banking APIs probably don't need to be hosted in the same API Management platform instance for Corporate clients banking APIs).

Another maybe a little bit less obvious example compared to API Management platform could be a Messaging platform. A pretty much everything said for API Management platform is applicable to Messaging platform with a difference that most (if not all) of the Messaging platform are lacking strong API catalogue like organizing and searching features but they are not too far from API Management platform in that sense.

Now maybe an examples where shared infrastructure or platform service is not a must.
If I have an aPaaS service and in this case I am not aware of underlying (virtual) servers maybe I don't need anymore to optimise the cost by hosting multiple application on the same set of (virtual) servers. Of course, who ever provides aPaaS service still deals with servers and manages these resources for the consumer but that is not the customer's problem any more - customer pays for what it uses.
Also, Operations wise, it is very debatable if there is any operational efficiency by hosting multiple applications within single aPaaS service.
Let's use Azure App Service for an example. App Service must use App Service Plan. Now if you make a parallel with pre PaaS era, you can say that App Service Plan is the same as Web Server farm and App Service is just an Web Site (which is actually not far from the truth). There is very little or no resource isolation between App Services hosted within the same App Service Plan. App Service plan has a "farm" of servers - virtual machine scale set. Scaling-out or up of the farm affects all App Services hosted within the App Service Plan.
There is very little Operations team has to really do on a regular bases in order to operate and maintain the App Service and App Service Plan. Apart form monitoring and having alerting configured most of not all of changes required to be performed within App Service is requested by Application team. Changes that are required across all of the App Services which may be originated by Operations team should be automated anyway - so applying the change on single App Service or on all App Services deployed by the organization is not (or it should not be) any more time wise expensive.
So I would argue that cost efficiency in the case of aPaaS should be a reason to always use aPaaS as a shared platform for multiple Applications.

# Line between Infrastructure or Platform service and Application Service

Given we deploy Application components onto a platform service where is the line between what is platform and what are application service owned components deployed to the platform?
This is not an article purely about API Management platform but this one is really a good example to describe this topic.
In order to publish an API, Application development team obviously needs to design it first and define an API contract typically described by using OpenAPI (Swagger) or some other API contract description standard (e.g. RAML, WADL, etc.). This API or sometimes also seen as a proxy of backend API belongs to an Application. Someone can argue that API itself could be an Application and this could be true in some cases but that doesn't change to point that an API either belong to an Application or it is in some cases Application itself and as such it is developed by Application development team.
Dilemma I saw often between Infrastructure / Platform teams and Application teams is, who should own the API given that an API has to be configured within the API Management platform? Infrastructure team would argue that since the platform requires specialist knowledge it is the best if specialised team takes ownership of configuring and deploying all of the APIs. It is certainly an option that will work but is it really the most scalable and does it provide desired Application development team independence I talked about earlier? Specialist knowledge - every new technology we introduce requires new knowledge. Complexity of our solutions is just growing and that is simply reality Application development team should be able to cope with. "Full stack developer" is a new norm and yes, API management or any other platform used to accelerate development simply has to be mastered by Application development team.
Alternative approach to the one where platform team takes responsibility to "configure" APIs on behalf / request of Application Development teams is to allow Application teams to use the platform and "develop" all aspects of an API on their own and simply use standard Continuous Delivery pipeline to deploy and release this API as part of their Application. Depending on the shared Platform service type, artefacts belonging to the Application are more or less isolated from other Application artefacts or from the platform itself. Specifically in case Azure API Management platform for instance, definition of the platform (configuration options) and API definitions are all part of the same Resource Management configuration. This may change in the future and it may be also different for some other platform services. There are platform configurations and there are configurations specific to Applications which are leveraging the platform and we should to be able to ensure that these can be managed independently within their own lifecycles and by providing isolation between these logical boundaries.

# Why separated lifecycle for centralized platform and applications is usually needed?

# Shared Platform and Application Service lifecycle management for Azure services

How to ensure independent platform and application lifecycles while keeping also high level of isolation between shared platform service and multiple Applications which are using the Platform?

Let's use different potentially shared platform service as example - Azure Service Bus as a messaging shared platform service.
In this scenario we may have three teams, two Application development teams and one shared platform service team. Each team owns certain components (or artefacts) which are physically part of the Azure Service Bus PaaS.
Both Application teams have a messaging topic each. They should define data structure of the messages in the topic, purpose of the Topic, external subscriber application access control for the topic, Topic behaviour like message retention period, etc
Shared platform service team which owns shared Azure Service Bus owns the instance itself and governs it's use. Specifically, they could be managing permissions for all of the environments (dev, test, production) for the logical instance, they are designing the platform based on required system quality attributes specified by Application teams (e.g. availability, etc), design and implementation of operational features like monitoring and alerting, etc
Therefore three different teams have different concerns, responsibilities and in many cases independent delivery timelines. 
In order to ensure team and application or service lifecycle independence all three teams should separate their Software Configuration Management including Continuous Delivery pipelines. How that concretely translate to the Azure Service Bus example?

Here is the breakdown of all artefacts for each team.

Service Bus centralized platform owner team:
- Shared platform definition configuration
    - Definition of the platform topology (e.g. multi-region deployment, service endpoint definition, etc)
    - ...

Application Team 1:
- "Topic 1" definition (Topic name, retention, access keys, partitioning scheme, etc)
- "Topic 1" subscription A definition (message filter definition, subscription access keys)
- Message A schema definition

Application Team 2:
- "Topic 1" definition (Topic name, retention, access keys, partitioning scheme, etc)
- "Topic 1" subscription A definition (message filter definition, subscription access keys)
- Message B schema definition

Here are several examples what separate ALM for each team also means and which tools and processes are separated:
- Separate Source Control repository
- Separate Continuous Delivery Pipeline
    - Build
    - Test Automation
    - Deployment Automation 

All team will have shared:
- Platform environments (Dev, Test, Production). Teams may chose to own their own Sandbox environment and be completely independent at that stage with freedom to experiment with different platform setups but sooner the teams start integrating their components / artefacts within the shared platform environment, the more chances teams will discover integration issues early in the process.

Should common System Testing for all three teams should be considered? Deploying Application 1 components (artefacts listed above) to the shared platform would ultimately mean that system testing is performed for both Application 1 and the platform. Unless two application teams have application dependencies there should be no need for these to participate in the common system test since platform service should provide enough runtime time isolation to prevent applications interfering with each other.

How we achieve independent Application component (artefact) deployment given that what we consider separated application component ultimately ends up incorporated into single Azure Resource - Azure Service Bus resource definition in this example?
Using Linked Resource templates (or resource dependencies or nested resources) is one the possible solutions. Definition of the component as Azure Resource as described in the breakdown above could be developed and maintained and deployed separately by each application team. 


[use ARM template example which specify all different aspects]



# Reviewers
...


# References

- (governance book about services, etc)
- Azure ARM template documents

Number of documents about creating Azure Resource Manager templates
[https://docs.microsoft.com/en-gb/azure/azure-resource-manager/](https://docs.microsoft.com/en-gb/azure/azure-resource-manager/)

Azure Policy helps you manage and prevent IT issues with policy definitions that enforce rules and effects for your resources. Use of Policies may be needed to enforce programmatically certain restrictions related to usage limits, allowed Azure technologies, etc
[https://docs.microsoft.com/en-gb/azure/azure-policy/](https://docs.microsoft.com/en-gb/azure/azure-policy/)

This article provides examples of how an enterprise can implement the recommendations for an Azure enterprise scaffold.
[https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/subscription-governance-examples](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/subscription-governance-examples)

General Azure as a Virtual Data Center guidance document. Broad list of topic including some relevant content related to resource isolation and separation of responsibilities.
[https://azure.microsoft.com/mediahandler/files/resourcefiles/1ad643b8-73f7-43f6-b05a-8e160168f9df/Azure_Virtual_Datacenter.pdf](https://azure.microsoft.com/mediahandler/files/resourcefiles/1ad643b8-73f7-43f6-b05a-8e160168f9df/Azure_Virtual_Datacenter.pdf)

Azure cloud governance refers to the decision-making processes, criteria, and policies involved in the planning, architecture, acquisition, deployment, operation, and management of cloud computing. Azure cloud governance provides an integrated audit and consulting approach for reviewing and advising organizations on their usage of the Azure platform
[https://docs.microsoft.com/en-us/azure/security/governance-in-azure](https://docs.microsoft.com/en-us/azure/security/governance-in-azure)