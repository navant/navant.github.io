---
title: "dbt-core vs dbt-cloud"
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
# dbt vs dbt core

dbt(a.k.a data build tool ) is the tool you common use for transforming your data in the modern data platform architecture. It that analysts and engineers transform data in their warehouse more effectively

their are two variants

- **dbt Core:** an open-source framework for transforming data (no cost)
- **dbt Cloud:** a managed service which provides Git-integrated code editing, job orchestration, and data quality controls on top of dbt Core’s transformation engine (SaaS license)

Now which one to use, i have seen most of the posts are driven by why dbt cloud, so this blog is about why not dbt Core, highlight whats needed  and when to use dbt cloud. 

Then you decide

| Feature | dbt Core | dbt Cloud | When to use dbt Cloud | What to do if dbt Core |
| --- | --- | --- | --- | --- |
| Local Development and IDE | Use VSCode or Pycharm on your desktop | Cloud IDE | if you dont want to go thru hassle of  setup your IDE in your workstation and install dbt etc . But most dev do have their IDE and their Python environment  | 1. You will use VSCode or Pycharm or others as IDE .                               2.Install Python and Python dependencies            3. install the dbt project 4. develop and run the project |
| CI/CD | Integrate to your github/git lab. need a docker/server and its connectivity for deployment | Integrate to your github/gitlab and you can execute from dbt cloud. still connectivity needed for deployment | if you dont have ci/cd or dont want to create hassle of ci/cd pipeline | setup a preinstalled docker with dbt packages , get the latest dbt project as part of ci/cd pipeline and execute |
| Documentation | dbt generate docs , this needs to be hosted as a static website  internally | dbt generate docs, its available in dbt cloud staightaway | if you dont want to host a static website based on ci/cd runs | host a static website at the end of ci/cd run. also this docs can be ingested into data catalog like alation, datahub |
| Scheduling and Orchestration | dbt projects needs to be executed on schedule or triggered  | dbt cloud has its own scheduler and also can integrate to airflow | if u need a scheduling and orchestrator | use airflow for scheduling and orchestration |
| metrics/semantic layer | dbt metrics available but deprecated. so this is open area (as of sept 2023) | use dbt metrics layer thats based on metrics flow | if you want central semantic layer,          (sth to explore: do u need it and why?) | use dbt metric or other semantic layer.  THIS IS KEY DIFFERENCE between dbt cloud vs dbtcore as dbt didnt upgraded dbt core with this feature.  but its unclear how a central metric/semantic layer will actual work and adoption is not clear.  |
| dbt APIs -metadata and cloud apis - Logging and Monitoring , Observability | elementary package can be installed to get this details | provides api to connect the dbt data to other down stream system | if you want metadata, data quality checks natively on the same ui | https://www.elementary-data.com/ , leverage slack connector.  create new dbt error log and integrate to cloud watch/splunk |

if you see above, most of the items can be done with dbt core with out the need for dbt cloud , but dbt cloud simplify a lot and suit smaller orgs to get started quickly

Considerations if you need connect to dbt cloud

- Connectivity to Databases
    - You will need to enable the dbt cloud connectivity to your data platform
    - in case of aws redshift or connecting to s3 its not straight forward ,you might need a bastion host to connect to redshift
        - [https://docs.getdbt.com/docs/cloud/connect-data-platform/connect-redshift-postgresql-alloydb](https://docs.getdbt.com/docs/cloud/connect-data-platform/connect-redshift-postgresql-alloydb)
- Prod deployment
    - You need prod deployment creds of your data warehouse  to be stored in dbt cloud to deploy to prod
- Connectiivity to your enterprise code repository
    - You need to connect to your organisation repo  [https://docs.getdbt.com/docs/cloud/git/connect-github](https://docs.getdbt.com/docs/cloud/git/connect-github)
- Considerations of how dbt is going to handle  Data
    - This has some limitations on where the cloud is actually hosted. Also some of the hosting options will be available only on Enterprise plans
- Access management and Logging
    - SSO is supported in enterprise version -
    - requires roles mapping and configuration
        - [https://docs.getdbt.com/docs/cloud/manage-access/about-user-access](https://docs.getdbt.com/docs/cloud/manage-access/about-user-access)
    - Audit logging is available in enterprise editon, Integration of audit logging need to be confirmed
- License, MSA etc
    - [3 Options - Developer, team and Enterprise.](https://www.getdbt.com/pricing)
    - if you need SSO , Audit logging or fine grain access controls(most enterprise do then you might need go to enterprise version)
        - is the security features be differentiator for different plans??-something for a different post.
- Others
    - Professional services, SLAs ,[https://www.getdbt.com/terms-of-service](https://www.getdbt.com/terms-of-service)
        - SLA is 3 9’s of reliability (99.9% -- allows 8 hours and 46 minutes of downtime per year.)
        - Support hours are 8 am - 8 pm

In my experience, it depends on enterprise, data retention and the enterprise security requirements, some of the above may be no-go in some enterprises or it takes longer to convince the security and architecture team to get this embedded into the organisation. so initially deploying dbt core is quick way forward and then later you can opt for dbt cloud(if still their is a advantage in dbt cloud)