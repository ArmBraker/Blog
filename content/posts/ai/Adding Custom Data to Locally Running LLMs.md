+++ 
draft = false
date = 2025-02-08T10:30:31+01:00
title = "Local LLMs Part 2: Adding Custom Data to Locally Running LLMs"
description = "Learn how to integrate custom data for LLMs"
slug = ""
authors = []
tags = [
    "LLM",
    "Ollama",
    "Machine Learning",
    "Retrieval-Augmented Generation (RAG)",
    "Ollama",
    "AnythingLLM",
]
categories = [
    "Local LLM Model",
    "Machine Learning",
]
externalLink = ""
series = ["Local LLM with Ollama"]
+++

{{< toc >}}

## Introduction

Welcome in the 2nd party of this series, in previous article [Introduction into LLM & Ollama Tool]([Local LLMs Part1: Introduction to LLM and Ollama · Journey into IT Knowledge](https://armbraker.github.io/blog/posts/ai/introduction-llm-part-1/))
we focused on short introduction to AI and LLM models their benefits and challenges.

In today article I would like to follow on the challenges which we can have with Public LLM models like ChatGPT or DeepSeek:

- **Data Privacy**
- **Cost**
- **Confidentiality**
- **Compliance and Regulations**

![](Pastedimage20250207134924.png)

I will try to summarize in this part how to feed LLMs with your data without loosing privacy and describe their *pros* and *cons* of these methods, including when to choose this method.

---

## Common Learning Methods

### 1. Fine-Tuning

| **Characteristic** | **Description** |
| --- | --- |
| **Definition** | Retraining a pre-existing LLM on your custom dataset to adapt it to specific tasks or domains. |
| **Pros** | Highly effective for domain-specific tasks, results in a more accurate and tailored model, can leverage pre-trained knowledge while adapting to new data. |
| **Cons** | Requires significant computational resources, time-consuming process, needs expertise in machine learning and access to high-quality datasets. |
| **When to Choose** | When you need a highly customized model for specific applications, when accuracy and domain-specific performance are crucial, when you have the necessary resources and expertise to fine-tune the model. |

### 2. Prompt Engineering

| **Characteristic** | **Description** |
| --- | --- |
| **Definition** | The art of creating well-crafted prompts that guide the LLM to produce desired outputs without altering the model itself. |
| **Pros** | Does not require retraining the model, quick and easy to implement, flexible and can be adjusted on the fly. |
| **Cons** | Limited by the inherent capabilities of the base model, may not perform as well on highly specialized tasks, can be less consistent compared to fine-tuned models. |
| **When to Choose** | When you need a quick solution without retraining the model, when dealing with general tasks that do not require extensive customization, when computational resources and time are limited. |

### 3. In-Context Learning

| **Characteristic** | **Description** |
| --- | --- |
| **Definition** | Providing examples within the prompt to guide the model's behavior for specific tasks. |
| **Pros** | No need for model retraining, allows for dynamic adjustment based on the task at hand, can improve performance with relevant examples. |
| **Cons** | Limited by the number of examples that can be provided within the prompt, effectiveness depends on the quality and relevance of the examples, may require frequent updates and adjustments. |
| **When to Choose** | When you need to quickly adapt the model to different tasks, when you have relevant examples readily available, when the task does not require extensive retraining. |

### 4. Retrieval-Augmented Generation (RAG)

| **Characteristic** | **Description** |
| --- | --- |
| **Definition** | Combining information retrieval with generation by allowing the LLM to access external documents or databases to generate more accurate and contextually relevant responses. |
| **Pros** | Provides up-to-date and relevant information from external sources, enhances the model's ability to answer specific questions accurately, reduces the need for extensive fine-tuning on large datasets. |
| **Cons** | Requires setting up and maintaining an external knowledge base or document repository, may introduce latency due to the retrieval process, needs additional components for effective retrieval and integration. |
| **When to Choose** | When you need the model to access a wide range of up-to-date information, when accuracy and context relevance are critical for the task, when you have the infrastructure to support document retrieval and integration.|

---

## Our Method of Choice

I personally prefer *Retrieval-Augmented Generation (RAG)* and *In-Context Learning* methods because they provide me the best results and keep my comfort 🛌 at high level.

There is too many ways how you can make RAG process tailored on your needs. The most common ways are by using Python 🐍 tools like [LangChain]([LangChain](https://www.langchain.com/)) with combination of vector database 📊 ([GitHub - chroma-core/chroma: the AI-native open-source embedding database](https://github.com/chroma-core/chroma))

However, I am very lazy person and I don't know much about the whole LLM and AI's internal workflows at the moment.  
Therefore, I decided to use easier way for the moment and use out of box solution [**AnythingLLM**](https://anythingllm.com/)

![](Pastedimage20250207141852.png)

>**AnythingLLM** is the easiest way to put powerful AI products like OpenAi, GPT-4, LangChain, PineconeDB, ChromaDB, and other services together in a neat package with no fuss to increase your productivity by 100x.

There are also other solutions and methods which can be used as LLM is very trendy topic  nowadays 🚀.
You may also check solutions like:

- [GPT4All](https://www.nomic.ai/gpt4all)
- [Open WebUI](https://openwebui.com/)
- [Build Your Own RAG App: A Step-by-Step Guide to Setup LLM locally using Ollama, Python, and ChromaDB - DEV Community](https://dev.to/nassermaronie/build-your-own-rag-app-a-step-by-step-guide-to-setup-llm-locally-using-ollama-python-and-chromadb-b12)

Maybe we will address some of them later. I also want to review RAG with LangChain and Python as it allows you to create your own pipelines.

## Technology Stack

Okay so, you’ve cut through all that text pile ⚔, we can finally start with deploying our solution with AnythingLLM and Ollama

![](Pastedimage20250207141810.png)

### Installation

Installation of Ollama was described already in my previous post  [Local LLMs Part1: Introduction to LLM and Ollama · Journey into IT Knowledge](https://armbraker.github.io/blog/posts/ai/introduction-llm-part-1/), and therefore I will not list it here and will continue only with installation and configuration of AnythingLLM.

Installation is very simple:

1. Visit their download page [Download AnythingLLM for Desktop](https://anythingllm.com/desktop)
2. Choose operation system (Windows, MAC or Linux)
3. Install
4. Done

***Note:*** Your browser show you a warning page as the file is not signed and other AVs can also have an issue to trust to this executable.

### Configuration

- **Disclaimer**: Agents functionality is not working with reasoning models like DeepSeek R1.

Once you have AnythingLLM installed, it is very easy to configure it utilize your local LLMs with Ollama.

1. Just choose provider Ollama in Search LLM provider window and it will identify you installation automatically:

![](Pastedimage20250207143904.png)
![](Pastedimage20250207143806.png)

2. You may leave advance settings as they are and continue.
3. You should see your configuration like:

![](Pastedimage20250207144028.png)

4. You can skip survey if you want.

## Provide Custom Data

### Local Data

1. Create workspace where you will store relevant data/documentation, e.g. "Blog".

![](Pastedimage20250207145708.png)

2. I don't know what is optimal way how to organize the data in the Workspaces, but I am creating folders and workspace according to topic which I want to provide to LLM. But I will leave it on everybody to figure out the a way working best for them.
3. You can upload your documentation and files by simple clink on upload icon next to name of your workspace

 ![](Pastedimage20250207150522.png)

4. Then select the files or simply drag and drop them, then click "move to workspace". They should appear in your workspace:

![](Pastedimage20250207150645.png)
![](Pastedimage20250207150506.png)

5. Simply as anything from your documentation:

![](Pastedimage20250207151230.png)

6. You may also check from here the LLM take the info:

![](Pastedimage20250207151328.png)

### Remote Data

**AnythingLLM** provide you a possibility to scrap data from Internet site or pull data from repositories (GitHub, GitLab, YouTube transcript) and import them into RAG.
**Note**: For Git solutions, you will need to obtain ***API*** key.

![](Pastedimage20250208094627.png)

After you scrap the website, it will create you a new folder in Documents section, and you will need to mark it to use in you **workspace**.

![](Pastedimage20250208094708.png)

Wualaaa, you can use this documents in your context window (chat) with AI. 🎉🥳

![](Pastedimage20250208101519.png)
