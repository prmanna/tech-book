---
title: "Parallelism Strategies in Deep Learning"
bookCollapseSection: true
weight: 22
---

### Introduction

Figure 8-1 depicts some of the model parameters that need to be stored in GPU memory: a) Weight matrices associated with connections to the preceding layer, b) Weighted sum (z), c) Activation values (y), d) Errors (E), e) Local gradients (local ∇), f) Gradients received from peer GPUs (remote ∇), g) Learning rates (LR), and h) Weight adjustment values (Δw).

In addition, the training and test datasets, along with the model code, must also be stored in GPU memory. However, a single GPU may not have enough memory to accommodate all these elements. To address this limitation, an appropriate parallelization strategy must be chosen to efficiently distribute computations across multiple GPUs.

This chapter introduces the most common strategies include data parallelism, model parallelism, pipeline parallelism, and tensor parallelism.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhxKwIJDZqBlcnIC2ZQ4eRWI296Ll2IztEZGB9eOGtqjwfP4wCYzY30LAxkwtdhXZIVRsaifBNT4ZutbUE380GOGRnetH2L3bI-5TDwTWxCQfC7n5ccbcpNdyOkM05LLOd4mzdbjhFrAuVG5lVJEcKwnjNMGgwfKSwoRzN52tW46VJn-opl5XchZ_4ny2w=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEhxKwIJDZqBlcnIC2ZQ4eRWI296Ll2IztEZGB9eOGtqjwfP4wCYzY30LAxkwtdhXZIVRsaifBNT4ZutbUE380GOGRnetH2L3bI-5TDwTWxCQfC7n5ccbcpNdyOkM05LLOd4mzdbjhFrAuVG5lVJEcKwnjNMGgwfKSwoRzN52tW46VJn-opl5XchZ_4ny2w)

  

**Figure 8-1:** _Overview of Neural Networks Parameters._

  

### Data Parallelism

  

In data parallelization, each GPU has an identical copy of the complete model but processes different mini-batches of data. Gradients from all GPUs are averaged and synchronized before updating the model. This approach is effective when the model fits within a single GPU’s memory.

In Figure 8-2, the batch of training data is split into eight micro-batches. The first four micro-batches are processed by GPU A1, while the remaining four micro-batches are processed by GPU A2. Both GPUs share the same model, and their input data are processed through all layers to generate a model prediction. The computation during the forward pass does not involve network load traffic. After computing the model error, the backpropagation algorithm starts the backward pass. The first step involves calculating the derivative of the model error, which is synchronized across the GPUs. Next, the error is propagated backward to calculate neuron-based errors, which are then used to compute gradients for each weight parameter. These gradients are synchronized across the GPUs.

The backpropagation algorithm running on GPUs then sums the gradients and divides the result by the number of GPUs. This averaged result is also synchronized across GPUs. In this way, during the backward pass, both local gradients and averaged gradients are synchronized.

In our simple two-GPU example, this process does not generate excessive network traffic, although the GPUs can use 100% of their NICs (network interface cards) forwarding capacity. However, if hundreds or even thousands of GPUs are used, the network traffic becomes significantly larger.

Inter-GPU network communication within a single server (using PCIe, NVLink) or between multiple servers (over InfiniBand, Ethernet, wireless) requires packet forwarding with minimal latency and in a lossless manner. Minimal latency is required to keep the training time as short as possible, while lossless transport is essential because training will pause if even a single packet is lost during synchronization. In the worst-case scenario, if no snapshot of the training progress is taken, the entire training process must be restarted from the beginning. Training a Large Language Model can take months or more.

Now, consider the electricity costs if training had already been running for two months and had to be restarted due to a single packet loss.

  

Power Consumption Example:

- A single GPU consumes roughly 350W under full load.
- Total power consumption for 50,000 GPUs: 350 W×50,000=17,500,000 W=17.5 MW
- For two months (60 days = 1,440 hours) of training: 17.5 MW×1,440 hours=25,200 MWh=25,200,000
- If electricity costs $0.10 per kWh, the total training cost will be: 25,200,000 kWh×0.10=2,520,000 USD

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhxxpzFnuLD9b5FnCoVbBeL7DrXdkLEyMUik1D5vSIRiQCdAqVFMFjb9IfmqeF1zBmrcsz_2FUyfkyH9QoxMYOeaEz8i5fY8_ipCdOuHWNWeRblyjY6VOhJQ-hvKcTOwA30yvuQHaQTcjU3nzekarketsv89CxLq3HUbIO5ugq3yPh13lZcCmwEP0E6LIA=w640-h360)](https://blogger.googleusercontent.com/img/a/AVvXsEhxxpzFnuLD9b5FnCoVbBeL7DrXdkLEyMUik1D5vSIRiQCdAqVFMFjb9IfmqeF1zBmrcsz_2FUyfkyH9QoxMYOeaEz8i5fY8_ipCdOuHWNWeRblyjY6VOhJQ-hvKcTOwA30yvuQHaQTcjU3nzekarketsv89CxLq3HUbIO5ugq3yPh13lZcCmwEP0E6LIA)

  
**Figure 8-2:** _Data Parallelism Overview._
* https://nwktimes.blogspot.com/2025/03/parallelism-strategies-in-deep-learning.html

**References:**
