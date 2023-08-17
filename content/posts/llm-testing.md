---
title: "Integrating LLM"
date: 2023-07-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["LLMs"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
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
    URL: "https://github.com/navant/content/posts"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
# Evaluation and Testing LLM Outputs

### 

This is  one of series of articles on different solution considerations for integrating into LLMs.

### Question

How can we evaluate and test the Large Language model outputs

This is critical question among enterprises for integrating LLMs into production

Their is no single answer here. Integrating to LLMs has been widespread already , their are lots of showcases, demo tutorials etc. However not many stories on integrating output response from LLMs directly into actions or to end users, especially when your business is at stake. I

So how can we ensure

the outputs are expected (or within the tolerance range )

within the guard rails

ethical and How can we log and observe this for improvement/feedback and as a proof of reference.

Lets cover what we know

### What are the toolsets available:

July 2023 (because changes so fast in this space)

Lang chain: Self-critique chain with constitutional AI

References:

[https://wandb.ai/a-sh0ts/ethical-ada-llm-comparison/reports/Applying-Ethical-Principles-Scoring-to-Langchain-Visualizing-with-W-B-Prompts](https://wandb.ai/a-sh0ts/ethical-ada-llm-comparison/reports/Applying-Ethical-Principles-Scoring-to-Langchain-Visualizing-with-W-B-Prompts--Vmlldzo0MTQxODQ2)