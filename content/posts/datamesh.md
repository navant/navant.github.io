---
title: "Data Mesh-my take aways"
date: 2023-09-12T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["data"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Complexities and Considerations"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page/
editPost:
    URL: "https://github.com/navant/navant.github.io/tree/main/content/posts"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
[Data Mesh -](https://martinfowler.com/articles/data-mesh-principles.html) When you plan to modernise your data landscape, you would have heard about this term many times . Their are plethora of articles , books, references  and community around this. Here i am going to write my takeways in this ,what is really hard or complex and is it right architecture for you

 If you like more comprehensive reading of the concept, you can skip and access [https://www.datamesh-architecture.com/](https://www.datamesh-architecture.com/) directly which is far more detailed. 

### What is Data mesh -

The term *data mesh* was coined by [Zhamak Dehghani](https://martinfowler.com/articles/data-mesh-principles.html)  in 2019 and is based on four fundamental principles that bundle well-known concepts.

They are

- **domain ownership** - principle: domain teams to take responsibility for their data
- **data as a product -**principle: a product thinking philosophy onto analytical data.
- **self-serve data infrastructure platform** -principle: adopt platform thinking to data infrastructure
- **federated governance-**principle: interoperability of all data products through standardization, which is promoted  by the central governance group
- **Any Other?**

![Untitled](/datamesh1.png)

<aside>
💡 First take away - Dont take this directly, these are key principles and your landscape may be different, so find out whats best for you. but before you reinvent a new principle , understand whether above  already address your problem or applicable for you

</aside>

So what problem it try to answer

- **the central data team often becomes a bottleneck**.
- **the  team has to discover and understand the necessary domain data**
- **domain teams own and know their domain**

Here the concept of domain is different among different organisations, is IT team federated along with business into seperate domain like crm, billing, order mgmt, finance etc. or its one IT and/or one BI team who maintain all the data?  . How you tailor the data mesh or this is not applicable for you?

their is no straight answer to this. So understand completely before you get into conclusions.

What are chracteristics of domain team? 

![Data Domains-Mesh characteristics.png](/datamesh2.png)

Is everything too logical , then how you implement this in engineering way below site explains in detail [https://www.datamesh-architecture.com/#tech-stacks](https://www.datamesh-architecture.com/#tech-stacks) . I dont want to reinvent the wheel

So lets get into whats harder, which is main aspect i want to cover .

> this is purely based on my experience , so this may be not a problem for you
> 

### **Complexities in Data Mesh implementation**

- Data mesh is cultural and organisation design than technical (like true agile project life cycle)
- You dont get too many data engineers nor the skill set in the organisation to do pure domain model or with many federated domain teams.
- Most of the implementations are mix of old and new stacks and with central team and some team wants to run independently on their own, so no green field
- Teams often dont give much importance to data catalog or metadata management,  some times you dont a proper data catalog, even if you have catalog , it is often not inline with actual data , Which data is relevant is most of times with SMEs. SMEs dont want to spend effort in keeping up Data Catalog upto date.
- When ever technical implementation of data mesh , you immediately create a producer , consumer and a governance node and try to manage the sharing policies via governance node. but practically this drift apart  because producer enable other access points to consumer than published means or consumer duplicate the data and do on own with little visibility
- Applying policies in automated way is hard.  Their is lot of techincal challenges in fully adopting central policy definition to federated policy enforcement.

### What i can adopt

It depends. you can pick whats best works for you., implement like ***Data Mesh Lite** if full data mesh is too much for you*

The concept of data product, data contracts and self-serve infrastructure,  all are vauable that can be accomodated even in central teams setup .

if you cant adopt true domain model. it is fine if single team can be owner of multiple domains or multiple data products as long as the principles of domain ownership and data product are propertly applied.

Reference:

[https://www.datamesh-architecture.com/](https://www.datamesh-architecture.com/)