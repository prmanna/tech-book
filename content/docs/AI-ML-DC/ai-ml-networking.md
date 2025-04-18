---
title: "AI/ML Networking"
bookCollapseSection: true
weight: 10
---

# Intro
### AI/ML Networking Part I: RDMA Basics
### AI/ML Networking: Part-II: Introduction of Deep Neural Networks
_Machine Learning (ML)_ is a subset of _Artificial Intelligence (AI)_. ML is based on algorithms that allow learning, predicting, and making decisions based on data rather than pre-programmed tasks. ML leverages _Deep Neural Networks (DNNs)_, which have multiple layers, each consisting of neurons that process information from sub-layers as part of the training process. _Large Language Models (LLMs)_, such as OpenAI’s GPT (Generative Pre-trained Transformers), utilize ML and Deep Neural Networks.

For network engineers, it is crucial to understand the fundamental operations and communication models used in ML training processes. To emphasize the importance of this, I quote the Chinese philosopher and strategist Sun Tzu, who lived around 600 BCE, from his work The Art of War.

> _If you know the enemy and know yourself, you need not fear the result of a hundred battles._

We don’t have to be data scientists to design a network for AI/ML, but we must understand the operational fundamentals and communication patterns of ML. Additionally, we must have a deep understanding of network solutions and technologies to build a lossless and cost-effective network for enabling efficient training processes.

In the upcoming two posts, I will explain the basics of: 

_a) Data Models:_ Layers and neurons, forward and backward passes, and algorithms. 

_b) Parallelization Strategies:_ How training times can be reduced by dividing the model into smaller entities, batches, and even micro-batches, which are processed by several GPUs simultaneously.

The number of parameters, the selected data model, and the parallelization strategy affect the network traffic that crosses the data center switch fabric.

After these two posts, we will be ready to jump into the network part. 

At this stage, you may need to read (or re-read) my previous post about Remote Direct Memory Access (RDMA), a solution that enables GPUs to write data from local memory to remote GPUs' memory.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/ai-ml-dc-1.jpg)
### AI/ML Networking: Part-III: Basics of Neural Networks Training Process
**References:**
