---
title: "Agent GPT handson"
date: 2023-08-26T11:30:03+00:00
draft: false
# weight: 1
# aliases: ["/first"]
tags: ["LLMs","agentgpt"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "Experience sharing with my handson on Agent GPT"
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
## Interact with your Gmail with Agent GPT
25/8/2023
### Idea:

Sometimes we signup for too many news letters and information digests across different form and give our email. Idea is to create summary from newsletters i received based on day/author and create a summary email send to you

### How i POCed it

leverage Gmail API and Langchain agent 

[https://python.langchain.com/docs/integrations/toolkits/gmail](https://python.langchain.com/docs/integrations/toolkits/gmail)

### Learning

- Getting gmail tkoen from credentials.json require some troubleshooting
- Summarise multiple emails usecase is not that straight forward
- You may try like read a particular email, create draft etc as interim usecases.

<aside>
💡 You will hit the token limit for the above usecases of reading email and summarise when u execute the above steps for the usecase.

</aside>

### To be continued …

### Troubleshooting

Getting the gmail access working is the hardest part. Credentials.json 

most of the time below step is failing

`from langchain.agents.agent_toolkits import GmailToolkit`

`toolkit = GmailToolkit()`

troubleshooting steps

remember to generatae the web_app not a desktop app when generating O-Auth 2.0 chain

you should try importing token using [https://github.com/googleworkspace/python-samples/blob/main/gmail/quickstart/quickstart.py](https://github.com/googleworkspace/python-samples/blob/main/gmail/quickstart/quickstart.py) and then retry above method with active session

make sure your [localhost](http://localhost) port is configured currently in the redirect_uri ([https://stackoverflow.com/questions/63647151/gmail-api-keeps-changing-the-localhost-address-python](https://stackoverflow.com/questions/63647151/gmail-api-keeps-changing-the-localhost-address-python))

Once above is good, then the entire flow of 

Create Gmail tool kit → Initialise Agent with LLM → Agent(Run) will work fine

Below are the gmail functions it has with langchain tools

- **GmailCreateDraft**
- **GmailSendMessage**
- **GmailSearch**
- **GmailGetMessage**
- **GmailGetThread**

### Classical error -

**InvalidRequestError**

: This model's maximum context length is 4097 tokens, however you requested XX tokens (4018 in your prompt; 256 for the completion). Please reduce your prompt; or completion length.

