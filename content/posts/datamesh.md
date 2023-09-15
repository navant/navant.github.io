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
description: "Complexity and Considerations"
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
In the journey to modernize your data landscape, you've likely come across the term "[Data Mesh -](https://martinfowler.com/articles/data-mesh-principles.html)" numerous times. There's a wealth of articles, books, references, and a thriving community dedicated to this topic. In this blog post, we'll delve into what Data Mesh really entails, explore its complexities, and determine if it's the right architectural approach for your organization.

If you prefer a more comprehensive resource on the concept, you can skip ahead and access [DataMesh-Architecture](https://www.datamesh-architecture.com/), which provides in-depth information.

## What is Data Mesh?

The term *Data Mesh* was coined by [Zhamak Dehghani](https://martinfowler.com/articles/data-mesh-principles.html) in 2019, building upon four fundamental principles that consolidate well-established concepts:

- **Domain Ownership:** This principle advocates that domain teams take ownership of their data.
- **Data as a Product:** It promotes a product-centric philosophy applied to analytical data.
- **Self-Serve Data Infrastructure Platform:** Embracing platform thinking when it comes to data infrastructure.
- **Federated Governance:** Encouraging interoperability among all data products through standardization, facilitated by a central governance group.

*(Are there any other principles you should consider?)*

![Data Mesh](/datamesh1.png)

> 💡 **First Takeaway:** It's important to note that these principles are not one-size-fits-all. Your specific landscape may require adjustments, so before crafting new principles, assess whether the existing ones can address your challenges and align with your needs.

Now, let's dive into the problems Data Mesh aims to solve:

- **The Central Data Team Bottleneck:** Often, the central data team becomes a bottleneck in data management.
- **Discovering and Understanding Domain Data:** Teams must discover and comprehend the data relevant to their domains.
- **Domain Teams Ownership and Knowledge:** Different organizations define domains differently. Is your IT team separated into domains like CRM, billing, order management, finance, etc., or is there a unified IT or BI team managing all data? Assess how Data Mesh aligns with your unique situation; there's no one-size-fits-all answer.

To truly grasp Data Mesh's implications, it's crucial to understand the characteristics of domain teams:

![Data Domains-Mesh Characteristics](/datamesh2.png)

If everything seems logical so far, you might wonder about the engineering implementation. Fortunately, you don't have to reinvent the wheel. [This site](https://www.datamesh-architecture.com/#tech-stacks) explains in detail how to implement Data Mesh from an engineering perspective.

Now, let's delve into the heart of the matter—what makes Data Mesh implementation challenging:

> Please note that these complexities are based on my personal experience and may not apply universally.

### Complexities in Data Mesh Implementation

1. **Cultural and Organizational Design:** Data Mesh is more about cultural and organizational design than just technical changes, similar to a true Agile project lifecycle.

2. **Limited Data Engineering Resources:** Finding sufficient data engineers with the required skill set within the organization can be a challenge.

3. **Mix of Old and New Stacks:** Most implementations involve a mix of legacy and modern technology stacks, often with central and independent teams, making it far from a greenfield project.

4. **Data Catalog and Metadata Challenges:** Teams sometimes overlook the importance of data catalog or metadata management. Even if a catalog exists, it may not align with actual data, as data relevance is often known by Subject Matter Experts (SMEs) who may not prioritize catalog upkeep.

5. **Challenges in Governance:** While technical implementations typically involve creating producer, consumer, and governance nodes to manage data sharing policies, these can diverge from the intended design. Producers may enable unintended access points, and consumers might duplicate data with limited visibility.

6. **Automated Policy Application:** Applying policies in an automated way can be challenging, and there are numerous technical hurdles in fully adopting central policy definitions for federated policy enforcement.

### What Can You Adopt?

The path you choose depends on your organization's unique circumstances. You can adopt what works best for you, even if a full Data Mesh implementation seems overwhelming. Consider a "Data Mesh Lite" approach, where you selectively implement elements like data products, data contracts, and self-serve infrastructure, even within a central team setup.

If a true domain model isn't feasible, it's acceptable for a single team to own multiple domains or data products, as long as the principles of domain ownership and data product management are correctly applied.


In conclusion, Data Mesh is a powerful concept, but its successful implementation requires a deep understanding of your organization's unique needs and challenges. By carefully considering its principles and adapting them to your context, you can leverage Data Mesh to transform your data landscape.

**Reference:** [DataMesh-Architecture](https://www.datamesh-architecture.com/)
