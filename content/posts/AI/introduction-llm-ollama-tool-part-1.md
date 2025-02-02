+++ 
draft = false
date = 2025-02-01T14:15:31+01:00
title = "Local LLMs Part1: Introduction to LLM and Ollama"
description = "Introducing LLM machine learning models and how to run them locally with the Ollama"
slug = "introduction-llm-part-1"
authors = []
tags = [
    "LLM",
    "Ollama",
    "Machine Learning",
]
categories = [
    "Local LLM Model",
    "Machine Learning",
]
externalLink = ""
series = ["Local LLM with Ollama"]
+++

## Introduction

Welcome to this new series focused on LLMs (Large Language Models) and running them locally using [Ollama](https://ollama.com/)).
Recently, I decided to take a little break from my usual domain of **Cybersecurity** to explore LLMs. This topic is very popular nowadays, and many believe it will impact every domain of our lives and work. I have used and tried various LLMs, such as OpenAI, Gemini, and Claude.
However, I have concerns about the privacy of my personal or company information, and I believe others share this concern. Therefore, I decided to delve deeper into this topic and learn about LLMs and how to ***"feed"*** them with personal information without worries of providing them into 3rd party.
![LLM Meme](blog/posts/AI/20250202110843.jpg)

## Content of the series 🧾

- **Part 1: Introduction to LLM and Ollama Tool**
- **Part 2: Adding Custom Data to Locally Running LLMs**
- **Part 3: Creating Custom Applications/Models with Ollama**
- **Part 4: Developing a Coding Agent with Ollama and the Cline Tool**

---

## **Part 1: Introduction to LLM and Ollama Tool**

### What are Large Language Models (LLMs)?

#### What are LLMs?

**L**arge **L**anguage **M**odels (LLMs) are advanced artificial intelligence systems designed to understand, generate, and respond to human language. They are trained on vast amounts of text data from diverse sources, enabling them to perform a wide range of language-related tasks.

There are also other variants of LLM, called **"Multimodal"** LLMs. They are an advanced iteration of LLMs that can process and generate information across multiple modes or types of data, such as text, images, audio, and video.

> In a nutshell, LLMs are designed to understand and generate text like a human, in addition to other forms of content, based on the vast amount of data used to train them. They have the ability to infer from context, generate coherent and contextually relevant responses, translate to languages other than English, summarize text, answer questions (general conversation and FAQs) and even assist in creative writing or code generation tasks. 🤖

#### Problems of LLM and AI 🔥

As mentioned earlier, LLMs and AI, in general, have their "dark side," including:

- **Data privacy** - This is a multi-level problem. Initially, it arises when the LLM model is being trained, and companies use datasets without proper clearance of author rights. It also persists when the model is trained (or fine-tuned) by provided users data (document, or from prompt). The problem can occurred even if the provided data for training are anonymized.
- **Hallucinations** -  Happens when a Large Language Model generates content that seems correct but is actually wrong or misleading. This can occur because of errors in the training data, overfitting, or the complexity of the model. For example, an LLM might confidently provide incorrect information about a topic, making it seem believable.
- **Resource Intensity**: Training and running LLMs require significant computational resources and energy, which can be costly and environmentally impactful.
- **Ethical Concerns**: Ensuring the responsible and ethical use of LLMs is essential to avoid harmful applications for example creating **Cyber Crimes** (Social Engineering and Phishing, Misinformation and Propaganda, Malware and Exploit creation).

![LLM meme 2](blog/posts/AI/20250202112843.jpg)

### What is Ollama?

![[Pasted image 20250202120505.png|300]]

Ollama is an **open-source** project that serves as a powerful and user-friendly platform for running LLMs on your local machine. This offers **enhanced privacy, cost savings, and offline access**. Ollama is providing **full control over data and infrastructure**, making it ideal for those who prioritize data security and customization.

> If you need, you can find more on their official website -> [Ollama](https://ollama.com/) or on their [GitHub - ollama/ollama: Get up and running with Llama 3.3, DeepSeek-R1, Phi-4, Gemma 2, and other large language models.](https://github.com/ollama/ollama)

### Running LLM Models Locally with Ollama

#### Choosing a Right Platform

I am using Windows as my base operating system, and LLMs run best with a GPU.
Therefore, I will guide you on how to use Ollama on Windows.

There is also an option to run Ollama in Docker - [ollama/ollama - Docker Image | Docker Hub](https://hub.docker.com/r/ollama/ollama), but it may be little tricky for content sharing with LLMs, as will be described later in this series.

**Note**: I am suggesting to follow their [GitHub](https://github.com/ollama/ollama) for installation instructions.

#### Installation

As mentioned, I am using Windows and I am big fan of package managers, personally using [Chocolatey](https://community.chocolatey.org/).

- To install Ollama with choco is very simple:

```Powershell
choco install ollama -y
```

![[Pasted image 20250202122026.png]]

#### Usage

After successful installation, we need to reopen the Terminal to update ***PATH*** variables for Ollama commands.:

```Powershell
Ollama -v
ollama version is 0.5.7
```

#### Local Model Installation

![Deepseak](blog/posts/AI/069ccc94-63b0-41e6-b2b3-e8e56068ab1a.webp | 300)
We will use ***DeepSeek-R1*** model, it is  DeepSeek’s first-generation reasoning models, achieving performance comparable to OpenAI-o1 across math, code, and reasoning tasks.

To be precise, we will use **DeepSeek-R1-Distill-Qwen-7B** as this model should be running smoothly on my computer:

- **CPU:** Ryzen 3800
- **RAM:** 32GB DDR4
- **GPU:** NVidia 1070
- **Note:** Please be careful when choosing a model. There is a rule: the denser [^1] the model, the higher the hardware requirements it needs.

##### Download the Model

```powershell
ollama run deepseek-r1:7b
```

![Downloading Model](blog/posts/AI/20250202124458.png)

##### Prompt

- Finally we have the Ollama installed and desired model downloaded, we can execute our first prompt. I used something a bit complex 👌:

> What is your name?

```markdown
>>> what is your name?
<think>

</think>

Greetings! I'm DeepSeek-R1, an artificial intelligence assistant created by DeepSeek. I'm at your service and would be
delighted to assist you with any inquiries or tasks you may have.
```

##### Usage

```text
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  stop        Stop a running model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information
```

- Example:
![Example](blog/posts/AI/20250202125919.png)

- Use `Ctrl + d` or `/bye` to exit.

[^1]: In this context, "***dense***" refers to the complexity and size of the model. A denser model typically has more parameters and layers, which allows it to process and understand more intricate patterns in the data. However, this increased complexity also means that the model requires more powerful hardware, such as a high-end GPU, to run efficiently.
