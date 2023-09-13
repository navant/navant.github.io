---
title: "# dbt: A Deep Dive into dbt-Core vs dbt-Cloud"
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
description: "dbt Core & dbt Cloud: Differences and Considerations"
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
[dbt](https://www.getdbt.com/) (Data Build Tool) is fast gaining traction in the realm of contemporary data platform architecture. Its primary function? To enable analysts and engineers to more efficiently transform data within their data warehouse.

The DBT universe comprises two main variants:

- **DBT Core:** This open-source framework is dedicated to data transformation and is available at no cost.
- **DBT Cloud:** A managed service that offers Git-integrated code editing, job orchestration, and data quality controls, all built on the robust engine of DBT Core. This comes under a SaaS licensing model.

A pressing question for many is, "Which one should I use?" Many discussions are towards the benefits of DBT Cloud. However, this article aims to provide a insights on the merits of DBT Core, provide the circumstances under which DBT Cloud might be more better, and finally, empower you with the knowledge to make an informed decision.

## DBT Core vs. DBT Cloud: Feature Breakdown

| Feature                                  | DBT Core | DBT Cloud | When to Choose DBT Cloud  | What to Do with DBT Core |
|------------------------------------------|----------|-----------|---------------------------|--------------------------|
| **Local Development & IDE**              | Employ VSCode or Pycharm on your desktop | Cloud-based IDE | When setting up an IDE on your workstation feels burdensome | 1. Use VSCode, Pycharm, or other IDEs. 2. Install Python and its dependencies. 3. Set up the DBT project. 4. Develop & run the project. |
| **CI/CD**                                | Requires integration with GitHub/GitLab, a Docker/server, and connectivity for deployment | Seamless integration with GitHub/GitLab and execution via DBT Cloud. Connectivity still essential for deployment | If establishing a CI/CD pipeline feels daunting | Use a Docker preloaded with DBT packages, fetch the latest DBT project during CI/CD runs, and execute. |
| **Documentation**                        | Generate documentation with DBT and host it internally as a static website | Automatically generate documentation accessible directly in DBT Cloud | To avoid setting up and hosting a static site | Generate a static site post CI/CD run. This documentation can also be integrated into data catalogs like Alation or DataHub. |
| **Scheduling & Orchestration**           | Manual scheduling or triggering of DBT projects is needed | Built-in scheduler and integration with tools like Airflow | When in-house scheduling and orchestration isn't feasible | Utilize Airflow for both scheduling and orchestration. |
| **Metrics/Semantic Layer**               | Available DBT metrics are now deprecated (as of Sept 2023) | Leverage the DBT metrics layer based on metrics flow | If a centralized semantic layer is vital. (Further exploration: Do you really need it?) | Employ DBT metric or another semantic layer. *Note:* This is a primary distinction between DBT Cloud and Core. The real-world functionality and uptake of a centralized metric/semantic layer remain ambiguous. |
| **APIs, Logging, Monitoring & Observability** | Install the elementary package for these insights | Offers APIs to connect DBT data to other downstream systems, ensuring unified monitoring | When desiring native metadata and data quality checks on a singular UI | Use [Elementary Data](https://www.elementary-data.com/), integrate with tools like Slack, and establish a DBT error log that can connect to CloudWatch or Splunk. |

From the table, it's evident that while many tasks can be accomplished using DBT Core alone, DBT Cloud offers a more streamlined experience, especially beneficial for smaller organizations aiming for a quick start.

## DBT Cloud: Key Considerations

Before diving into DBT Cloud, here are some pivotal aspects you need to mull over:

- **Database Connectivity:** Ensure DBT Cloud can interface with your data platform. For platforms like AWS Redshift or S3, the connection might necessitate a bastion host. [Learn more](https://docs.getdbt.com/docs/cloud/connect-data-platform/connect-redshift-postgresql-alloydb).
- **Production Deployment:** DBT Cloud requires your data warehouse's production deployment credentials for a seamless transition to production.
- **Repository Connectivity:** Establish a connection to your organization's code repository. [Here's how to connect to GitHub](https://docs.getdbt.com/docs/cloud/git/connect-github).
- **Data Handling by DBT:** Some limitations exist based on where the cloud is hosted. Additionally, specific hosting options are exclusive to Enterprise plans.
- **Access Management & Logging:** While SSO is available in the enterprise version, it demands role mapping and configuration. Audit logging, also an enterprise feature, requires verification for integration. [More on user access](https://docs.getdbt.com/docs/cloud/manage-access/about-user-access).
- **Licensing:** DBT Cloud offers [three distinct plans](https://www.getdbt.com/pricing): Developer, Team, and Enterprise. Should your enterprise need features like SSO, Audit logging, or granular access controls, the Enterprise version might be your best bet. Is security a differentiator across plans? That's a topic for another day.
- **Additional Factors:** Professional services, SLAs, and other terms of service are crucial. For instance, DBT Cloud promises 99.9% reliability, translating to a downtime of 8 hours 46 minutes annually. Support is available from 8 am to 8 pm.

## Final Thoughts

From my vantage point, the choice between DBT Core and Cloud largely circulate on an enterprise's data retention policies, security prerequisites, and the organization's readiness to adopt new tools. In some cases, stringent security protocols might make the adoption of DBT Cloud challenging. Starting with DBT Core can provide a swift entry point, and as the organization evolves, transitioning to DBT Cloud might become a logical next step — provided it still offers a distinct advantage.
