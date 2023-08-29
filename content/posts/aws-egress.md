---
title: "AWS Egress cost"
date: 2023-08-29T11:30:03+00:00
draft: false
# weight: 1
# aliases: ["/first"]
tags: ["cloud-cost","aws"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "Beware of Inter-region aws data transfer costs"
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
    URL: "https://github.com/navant/navant.github.io/tree/main/content/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
29/8/2023

Their is a common misconception that AWS Egress cost to internet is expensive( and it is) but did you know cross-region is not cheap either

I have based this calculation based on AWS Sydney region as base region (Aug 29th 2023)

via the cost calculator - for data transfer charges [https://calculator.aws/#/estimate](https://calculator.aws/#/estimate)

| Region | Transfer to | Data Amount | Cost Estimate per Month |
| --- | --- | --- | --- |
| AWS Sydney region | Internet | First 100 GB | FREE |
| AWS Sydney region | Internet | 1 TB | 132 USD |
| AWS Sydney region | AWS Melbourne Region | 1 TB | 102 USD |
| AWS Sydney region | Other Region | 1 TB | 120.83 USD |
| AWS Sydney region | CloudFront | 1 TB | Free |

How about Ingress Cost - Ingress transfer cost is FREE (aws wants your data in)