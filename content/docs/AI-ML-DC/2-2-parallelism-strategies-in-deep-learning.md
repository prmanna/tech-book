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


### Model Parallelism with Pipeline Parallelism

In **Model Parallelism**, the neural network is partitioned across multiple GPUs, with each GPU responsible for specific layers of the model. This strategy is particularly beneficial for large-scale models that surpass the memory limitations of a single GPU.

Conversely, **Pipeline Parallelism** involves dividing the model into consecutive stages, assigning each stage to a different GPU. This setup allows data to be processed in a pipeline fashion, akin to an assembly line, enabling simultaneous processing of multiple training samples. Without pipeline parallelism, each GPU would process its inputs sequentially from the complete dataset, while all other GPUs remain idle.

Our example neural network in Figure 8-3 consists of three hidden layers and an output layer. The first hidden layer is assigned to **GPU A1**, while the second and third hidden layers are assigned to **GPU A2** and **GPU B1**, respectively. The output layer is placed on **GPU B2**. The training dataset is divided into four micro-batches and stored on the GPUs. These micro-batches are fed sequentially into the first hidden layer on GPU A1. 

**_Note 8-1._** _In this example, we use a small training dataset. However, if the dataset is too large to fit on a single GPU, we combine model parallelism, pipeline parallelism, and data parallelism to distribute the workload efficiently. See the note 8-2 for more detail._

I have divided the forward pass and backward pass into time steps, which are further split into computation and communication phases.

During the forward pass, neurons first calculate the weighted sum of inputs, apply the activation function, and produce an output y (computation phase). The computed outputs y, stored in GPU memory, are then transferred to peer GPU(s) using Remote Direct Memory Access (RDMA) (communication phase).

During the backward pass, the backpropagation algorithm computes the model error (computation phase) and propagates it backward across GPUs using RDMA (communication phase). This process was explained in detail in Chapter 2. 

**_Note 8-2:_** _In our example, Hidden Layer 1 fits entirely on GPU A1. The same applies to other layers—they each fit within a single GPU. However, if the input dataset is too large to fit into a single GPU, it must be split across multiple GPUs. In that case, Hidden Layer 1 will be distributed across multiple GPUs, with each GPU handling a different portion of the dataset. When this happens, the gradients of Hidden Layer 1 must be synchronized across all GPUs that store part of the layer._

_Time step 1:_

     **Computing:**

·    A1 processes the input x1 resulting the output y1.

**Communication:**

·    A1 transports y1 to A2. 

**Active GPUs (25%):** A1

_[![](https://blogger.googleusercontent.com/img/a/AVvXsEgPEPTUr3eU_adLUyu1nUOVNPzcXhEnrQDO_tNOg-IJIQerfKcS_DIeI6RUOBR7SUe8_srgnt5CD04eoqtDKlO30EL-_Vn6Kru-egMcxDpzFuIyysrmvEny5GibdYYk7K040_u24vscJ84GF7LKJRziC26rUm6YPliDZMC8JUxVolFpw5mof0qk9E2rX2o=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEgPEPTUr3eU_adLUyu1nUOVNPzcXhEnrQDO_tNOg-IJIQerfKcS_DIeI6RUOBR7SUe8_srgnt5CD04eoqtDKlO30EL-_Vn6Kru-egMcxDpzFuIyysrmvEny5GibdYYk7K040_u24vscJ84GF7LKJRziC26rUm6YPliDZMC8JUxVolFpw5mof0qk9E2rX2o)_

**Figure 8-3:** _Model Parallelism with Pipeline Parallelism – Time Step 1._

_Time step 2:_ 

**Computing:**

·    A1 processes the input x2 and produces the output y2.

·    A2 processes the input y1 and produces the output y1.  
  

**Communication:**

·    A1 transports y2 to A2.

·    A2 transports y1 to B3. 

**Active GPUs (50%):** A1, A2

  

_[![](https://blogger.googleusercontent.com/img/a/AVvXsEhrzoBl7grP-8SVcxpaqJH0sxJvjDqlQNJoFXPC2TbdYNmE8UnOm4QhEKV4Cv8PIqH_u9fogdFr7yNfXTFiWcbxHEcx70U6YGaXxkRJmbbIlLYBDGNtlm_F-JZVTTbi_BMKgG5wZj16EpkNDHrYkkJ49obm3YSRHismFZLzx1YgCuF5edh8q1llXSm8lmU=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEhrzoBl7grP-8SVcxpaqJH0sxJvjDqlQNJoFXPC2TbdYNmE8UnOm4QhEKV4Cv8PIqH_u9fogdFr7yNfXTFiWcbxHEcx70U6YGaXxkRJmbbIlLYBDGNtlm_F-JZVTTbi_BMKgG5wZj16EpkNDHrYkkJ49obm3YSRHismFZLzx1YgCuF5edh8q1llXSm8lmU)_

**Figure 8-4:** _Model Parallelism with Pipeline Parallelism – Time Step 2._

_Time step 3:_

**Computing:**

·    A1 processes the input x3 and produces the output y3.

·    A2 processes the input y2 and produces the output y2.

·    B1 processes the input y1 and produces the output y1.  
  

**Communication:**

·    A1 transports y3 to A2.

·    A2 transports y2 to B1.

·    B1 transports y1 to B2

 **Active GPUs (75%):** A1, A2, B1

_[![](https://blogger.googleusercontent.com/img/a/AVvXsEhNtFfIjX1JB4I4ze0LQ3B-cMG1JpdtPAYc4ydhYeFyhOXC_Agio0sRDpxc40JZxuhAyk9DTHt1YzxmV6IzSimqIFH8F61OryTSoIYG1hB4_2vj1mrOHFpodkBrnTeEhbDhtsBl4zmGLFSqIrb15WxNLVzgFTPkr5W21qhnruNau2-Ee5v8wdIm3yV-kFU=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEhNtFfIjX1JB4I4ze0LQ3B-cMG1JpdtPAYc4ydhYeFyhOXC_Agio0sRDpxc40JZxuhAyk9DTHt1YzxmV6IzSimqIFH8F61OryTSoIYG1hB4_2vj1mrOHFpodkBrnTeEhbDhtsBl4zmGLFSqIrb15WxNLVzgFTPkr5W21qhnruNau2-Ee5v8wdIm3yV-kFU)_

**Figure 8-5:** _Model Parallelism with Pipeline Parallelism – Time Step 3._ 

_Time step 4:_ 

**Computing:**

·    A1 processes the input x4 and produces the output y4.

·    A2 processes the input y3 and produces the output y3.

·    B1 processes the input y2 and produces the output y2.

·    B2 processes the input y1 and produces the model output 1.  
  

**Communication:**

·    A1 transports y3 to A2.

·    A2 transports y2 to B1.

·    B1 transports y1 to B2 

**Active GPUs (100%):** A1, A2, B1, B2

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgDoOcf55OcJz0amqNyGFcD5Mg2oHel5M6D-usGKlBKoP9daJhAd_0O8t2DMlKccRAjmtqAgu5LN6rrURW2lEYUfvmd48GA7_Z4O9hRU-Ct6hjWCoJzSydQ_sBpxYxKkjUi2-5hmUQODWuJ6mXKZhXDekH8tegFHehxIil_7taKP0DSG8jJbe8KlnVMPOM=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEgDoOcf55OcJz0amqNyGFcD5Mg2oHel5M6D-usGKlBKoP9daJhAd_0O8t2DMlKccRAjmtqAgu5LN6rrURW2lEYUfvmd48GA7_Z4O9hRU-Ct6hjWCoJzSydQ_sBpxYxKkjUi2-5hmUQODWuJ6mXKZhXDekH8tegFHehxIil_7taKP0DSG8jJbe8KlnVMPOM)

  

**Figure 8-6:** _Model Parallelism with Pipeline Parallelism – Time Step 4._ 

_Time step 5:_

**Computing:**

·    A2 processes the input y4 and produces the output y4.

·    B1 processes the input y3 and produces the output y3.

·    B2 processes the input y2 and produces the model output 2.

·    B2 Computes local neuron error E1, and gradient G1.  
  

**Communication:**

·    A2 transports y4 to B1.

·    B1 transports y3 to B2.

·    B2 transports error E1 to B1

**Active GPUs (75%):** A2, B1, B2

 The notation x3 above G1 on GPU B2 indicates that the algorithm computes gradients from the error for each weight associated with the inputs, including the bias. This process is repeated with all four micro-batches. This notation will be used in the upcoming figures as well.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEi7KUd9simQLOfwbkpl51rMjJWG4LV2zqNKSW89d93HB3ee0nZGAZx8Dd3MFufLf9V6iP4uozupMZybimvNWe5dsQQV_Gpr7n8vLT7wcBcsYTyheCkbKznkRCbIIX-RKzUWbWdRM1lRFMWCB1upnIQdeQqqBJ3uncW4cgLtrnSvryCg5BK-z9etcdITNNw=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEi7KUd9simQLOfwbkpl51rMjJWG4LV2zqNKSW89d93HB3ee0nZGAZx8Dd3MFufLf9V6iP4uozupMZybimvNWe5dsQQV_Gpr7n8vLT7wcBcsYTyheCkbKznkRCbIIX-RKzUWbWdRM1lRFMWCB1upnIQdeQqqBJ3uncW4cgLtrnSvryCg5BK-z9etcdITNNw)

  

**Figure 8-7:** _Model Parallelism with Pipeline Parallelism – Time Step 5._ 

_Time step 6:_ 

**Computing:**

·    B1 processes the input y4 and produces the output y4.

·    B2 processes the input y3 and produces model output 3.

·    B2 Computes local neuron error E2, and gradient G2.

·    B1 Computes local neuron error E1, and gradient G1.  
  

**Communication:**

·    B1 transports y4 to B2.

·    B2 transports error E2 to B1

 **Active GPUs (50%):** B1, B2

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgBZnMvuOTvtGmPpFh4sfv5mN2jEVj6M4lMTLF3pX2IFF8CViAipnLD4RWC1g61tNzpO4Kpv204bpuPmS6Kfl08YqqRORYnkCYTjGRKnm3ce62Kt_PGyr0NQqwhtsPNufVthqj8RoyR8G7CgGeQSbGOCpsXKjmThRNh7j7Hvy87viqAfdck42mWddMAxcc=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEgBZnMvuOTvtGmPpFh4sfv5mN2jEVj6M4lMTLF3pX2IFF8CViAipnLD4RWC1g61tNzpO4Kpv204bpuPmS6Kfl08YqqRORYnkCYTjGRKnm3ce62Kt_PGyr0NQqwhtsPNufVthqj8RoyR8G7CgGeQSbGOCpsXKjmThRNh7j7Hvy87viqAfdck42mWddMAxcc)

  

 **Figure 8-8:** _Model Parallelism with Pipeline Parallelism – Time Step 6._ 

_Time step 7:_ 

**Computing:**

·    B2 processes the input y4 and produces model output 4.

·    B2 Computes local neuron error E3, and gradient G3.

·    B1 Computes local neuron error E2, and gradient G2.

·    A2 Computes local neuron error E1, and gradient G1.  
  

**Communication:**

·    B2 transports error E3 to B1

·    B1 transports error E2 to A2

·    A2 transports error E1 to A1 

**Active GPUs (75%):** A2, B1, B2

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhwEw6vItig1kkgwBoQsnlmg4tuobhOPEzvQyw7ZGd7h8j5T0cd_GeW1YxZG19C0PfwMHEQ0eAhWzKOLd0OKwSaoDPWDFghNfDeZCvH9yGxCs2cVz2jFyNpRirSMwehaS-TkJWISNcxTj_s-CK9p3VN-f5-jkMxJIJmeqkZcxBmEmWmnrjNyPpF4qRvjwk=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEhwEw6vItig1kkgwBoQsnlmg4tuobhOPEzvQyw7ZGd7h8j5T0cd_GeW1YxZG19C0PfwMHEQ0eAhWzKOLd0OKwSaoDPWDFghNfDeZCvH9yGxCs2cVz2jFyNpRirSMwehaS-TkJWISNcxTj_s-CK9p3VN-f5-jkMxJIJmeqkZcxBmEmWmnrjNyPpF4qRvjwk)

**Figure 8-9:** _Model Parallelism with Pipeline Parallelism – Time Step 7._ 

_Time step 8:_ 

**Computing:** 

·    B2 Computes local neuron error E4, and gradient G4.

·    B1 Computes local neuron error E3, and gradient G3.

·    A2 Computes local neuron error E2, and gradient G2.

·    A1 Computes local neuron error E1, and gradient G1.  
  

**Communication:**

·    B2 transports error E4 to B1

·    B1 transports error E3 to A2

·    A2 transports error E2 to A1 

**Active GPUs (100%):** A1, A2, B1, B2

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjmCDmYvEVRBWO6_9mFkGy5ELIRNmCFQMZ4X1yX3gdV7v3Pa5V_Q6BSL4HE_sgf31hJwRXpWeG9X333_hmdgP4xsaiec4EHWuHjuvvbFq2Nc6MTGM5-3glfHbka0MOaA0Xeyhhrtzp8RrHlDzb9uVC2M5g4GiWSfUKTXRDj97vuLcJydliWGyyOVikQF2I=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEjmCDmYvEVRBWO6_9mFkGy5ELIRNmCFQMZ4X1yX3gdV7v3Pa5V_Q6BSL4HE_sgf31hJwRXpWeG9X333_hmdgP4xsaiec4EHWuHjuvvbFq2Nc6MTGM5-3glfHbka0MOaA0Xeyhhrtzp8RrHlDzb9uVC2M5g4GiWSfUKTXRDj97vuLcJydliWGyyOVikQF2I)

**Figure 8-10:** _Model Parallelism with Pipeline Parallelism – Time Step 8._ 

_Time step 9:_ 

**Computing:** 

·    B1 Computes local neuron error E4, and gradient G4.

·    A2 Computes local neuron error E3, and gradient G3.

·    A1 Computes local neuron error E2, and gradient G2.  
  

**Communication:** 

·    B1 transports error E4 to A2

·    A2 transports error E3 to A1 

**Active GPUs (75%):** A1, A2, B1

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgDuyIjMTbYZ7x-viIHo_OwbksQ3IxeJy_otktX-wLVHsKxUdzHCrolZPEd1hQ5RMd42yTwjZwf_k69TFiwMuPXXV29KYPqrsB2yFjaQ3zJCZFv5_Gm7uv6rHYucpqsr0LK45b92PJp13VNRHOefrIpRC7DB_XYnGg2dGVWczFzUn5qIYXgere9-Mjhuwk=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEgDuyIjMTbYZ7x-viIHo_OwbksQ3IxeJy_otktX-wLVHsKxUdzHCrolZPEd1hQ5RMd42yTwjZwf_k69TFiwMuPXXV29KYPqrsB2yFjaQ3zJCZFv5_Gm7uv6rHYucpqsr0LK45b92PJp13VNRHOefrIpRC7DB_XYnGg2dGVWczFzUn5qIYXgere9-Mjhuwk)

**Figure 8-11:** _Model Parallelism with Pipeline Parallelism – Time Step 9._ 

_Time step 10:_

**Computing:** 

·    A2 Computes local neuron error E4, and gradient G4.

·    A1 Computes local neuron error E3, and gradient G3.  
  

**Communication:** 

·    A2 transports error E4 to A1 

**Active GPUs (50%):** A1, A2

[![](https://blogger.googleusercontent.com/img/a/AVvXsEimhrYZCYa895nZKIVTgKhI_y0iho3m981oRsp2caO778jRJQszzdgchQI7I57pWgkS5l98kSiA2yn5rlCoZGukOKB9nE0vQi3rB6OuIVlFRXAfQANyGsQ_7CD8rVA4AQ5m2nou-Sr39apMLl1HrLL3g_Nd22d68MIQwJbNzZpeaEYRT5wUx1irJvsCSVE=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEimhrYZCYa895nZKIVTgKhI_y0iho3m981oRsp2caO778jRJQszzdgchQI7I57pWgkS5l98kSiA2yn5rlCoZGukOKB9nE0vQi3rB6OuIVlFRXAfQANyGsQ_7CD8rVA4AQ5m2nou-Sr39apMLl1HrLL3g_Nd22d68MIQwJbNzZpeaEYRT5wUx1irJvsCSVE)

**Figure 8-12:** _Model Parallelism with Pipeline Parallelism – Time Step 10._ 

_Time step 11:_ 

**Computing:** 

·    A1 Computes local neuron error E4, and gradient G4.  
  

**Communication:** 

**Active GPUs (25%):** A1

In our example, the micro-batches fit into a single GPU, so we don’t need to split them across multiple GPUs. That said, once GPU A1 has computed the gradients for the last micro-batch, the weights are adjusted, and the second iteration of the forward pass begins.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEj3HmVW1cfX-LMEovgxa8uMfjC72doqxZNvHhjKALerlbfRKmKaBuOgwNDxbBGadHPqZWHPZm_uWQ_ioWeKYqHzRF2dLG-KnhodmQeSfZePENdxH84LDEaBuISfNxP5Crsn2-NhMAbseYzKyznifQjLUDi8WI2oc5E-jwSeGNbDKbw90yEM27TM03XOoyk=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEj3HmVW1cfX-LMEovgxa8uMfjC72doqxZNvHhjKALerlbfRKmKaBuOgwNDxbBGadHPqZWHPZm_uWQ_ioWeKYqHzRF2dLG-KnhodmQeSfZePENdxH84LDEaBuISfNxP5Crsn2-NhMAbseYzKyznifQjLUDi8WI2oc5E-jwSeGNbDKbw90yEM27TM03XOoyk)

**Figure 8-13:** _Model Parallelism with Pipeline Parallelism – Time Step 11._ 

If the test dataset is too large for a single GPU and must be split across multiple GPUs, the layers must also be shared between GPUs. For example, hidden layer 1 is on GPUs A1 and C1, while hidden layer 2 is on GPUs A2 and C2. This requires intra-layer gradient synchronization between GPUs sharing the same layer, resulting in inter-GPU packet transport. Figure 8-14 illustrates how gradients are first synchronized (inter-layer). Then, each GPU averages the gradients (sum of gradients divided by the number of GPUs). Finally, the averaged gradients are synchronized.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhLQP9-0LEL8YJrs6sJm30xBLyKcHc4g3OzYyuV-6Br0JmunsExlDxgu2LtcghPwuAIVB-fU5doawgrYhPX7f3D9r02r0JmwyNW7Ph3ItzxF5oS3l9JcbZOOS-Yz6QkSsDa277xqxFAC8i4eb6CHpicbAZEAE1ooG9rfwR129YakHDjFXHBNkslzTVJcDM=w640-h366)](https://blogger.googleusercontent.com/img/a/AVvXsEhLQP9-0LEL8YJrs6sJm30xBLyKcHc4g3OzYyuV-6Br0JmunsExlDxgu2LtcghPwuAIVB-fU5doawgrYhPX7f3D9r02r0JmwyNW7Ph3ItzxF5oS3l9JcbZOOS-Yz6QkSsDa277xqxFAC8i4eb6CHpicbAZEAE1ooG9rfwR129YakHDjFXHBNkslzTVJcDM)

**Figure 8-14:** _Model Parallelism with Pipeline Parallelism – Synchronization._

### Tensor Parallelism

 The previous section described how Pipeline Parallelism distributes entire layers across multiple GPUs. However, Large Language Models (LLMs) based on transformer architectures contain billions of parameters, making this approach insufficient.

For example, GPT-3 has approximately 605 million parameters in a single self-attention layer and about 1.2 billion parameters in a feedforward layer, and these figures apply to just one transformer block. Since GPT-3 has 96 transformer blocks, the total parameter count reaches approximately 173 billion. When adding embedding and normalization parameters, the total increases to roughly 175 billion parameters.

The number of parameters in a single layer alone often exceeds the memory capacity of a single GPU, making Pipeline Parallelism insufficient. Additionally, performing large matrix multiplications on a single GPU would be extremely slow and inefficient. Tensor Parallelism addresses this challenge by splitting computations within individual layers across multiple GPUs rather than assigning whole layers to separate GPUs, as done in Pipeline Parallelism.

Chapter 7 introduces Transformer architecture but for memory refreshing, figure 8-15 illustrates a stack of decoder modules in a transformer architecture. Each decoder module consists of a Self-Attention layer and a Feedforward layer. The figure also shows how an input word, represented by x1, is first mapped to a token. The token, in turn, receives a positional word embedding vector through lookups in the word embedding and position embedding tables.

The resulting word vector is used to compute Query (Q) and Key (K) matrices, which, in turn, produces logits via dot products. These logits are then passed through the SoftMax function. The resulting matrix from the SoftMax function is multiplied with the Value (V) matrices. After Add & Normalization computation, the resulting matrix is fed into the Feedforward, fully connected, neural network.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhuiHOC2YVNBLlpLcNtL6gGj_ITIiXiF5pirOTinoCyOS_RLDerUs85M1VifJPdSZHJgVvYesvn5uQFr5ZPhWHqRS1_N5VjXg3rOKfFy1JUbsjTwpxm3gqyQfcifm-5gai05EQWKZzGkjaTvBoi3L2lzXY7ZvAa1T8Aj9HQuYXqGfI6d0n8pe8jyv_naaA=w640-h382)](https://blogger.googleusercontent.com/img/a/AVvXsEhuiHOC2YVNBLlpLcNtL6gGj_ITIiXiF5pirOTinoCyOS_RLDerUs85M1VifJPdSZHJgVvYesvn5uQFr5ZPhWHqRS1_N5VjXg3rOKfFy1JUbsjTwpxm3gqyQfcifm-5gai05EQWKZzGkjaTvBoi3L2lzXY7ZvAa1T8Aj9HQuYXqGfI6d0n8pe8jyv_naaA)

**Figure 8-15:** _An Overview of a Transformer Architecture._

  
### Self-Attention Layer

In most cases, the word embedding matrix fits within a single GPU and is not split across multiple GPUs when using Tensor Parallelism. This is because a typical embedding matrix is approximately 200 MB, which is significantly smaller than large Transformer layers that can contain billions of parameters.

  

Another reason for keeping the embedding matrix on a single GPU is efficient lookup operations. Unlike large matrix multiplications, embedding lookups are memory-efficient and do not impose significant computational overhead. Splitting the embedding matrix across multiple GPUs would introduce high communication costs, as each GPU would store only a fraction of the vocabulary. This would require frequent cross-GPU communication for token lookups, increasing latency and reducing efficiency. After the embedding lookup, the embedding vectors are broadcasted to all GPUs before the Transformer computations start. 

  

However, in very large-scale models (such as GPT-3 with 175 billion parameters), embeddings may be sharded across multiple GPUs using distributed embeddings or model parallelism techniques. One approach is row-wise parallelism, where the vocabulary is split across GPUs, and each GPU stores only a fraction of the embeddings, handling lookups for the tokens it owns. 

  

Figure 8-16 illustrates how the positional word embedding matrix (Eepv) is multiplied with the Query (Q), Key (K), and Value (V) matrices to produce the corresponding Q, K, and V vectors. The Query and Key vectors are then used as inputs to the self-attention layer.

  

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgaQmUUCroHyFYxNQfkn93h33VlST_C05Su8Cf-p1ZsK8NqKKMi87bNXjQvE0cgzLAu34LKP1GZvpKj9HFmhid0L8fx63smIcL18TZpXdwYuHIr43pR_QSnqg_zFvJlkVsV401JAjhhr0Zt3b__nlR6_xfNGkUsT_SprQz6MdqsB3VT0S4rRyTbw84rtN4=w640-h194)](https://blogger.googleusercontent.com/img/a/AVvXsEgaQmUUCroHyFYxNQfkn93h33VlST_C05Su8Cf-p1ZsK8NqKKMi87bNXjQvE0cgzLAu34LKP1GZvpKj9HFmhid0L8fx63smIcL18TZpXdwYuHIr43pR_QSnqg_zFvJlkVsV401JAjhhr0Zt3b__nlR6_xfNGkUsT_SprQz6MdqsB3VT0S4rRyTbw84rtN4)

**Figure 8-16:** _Local Query (Q), Key (K), and Value (V) Matrices._

  

  

Figure 8-17 illustrates how the Query, Key, and Value matrices are sharded across two GPUs. The first fragments of these matrices are assigned to GPU A1, while the second fragments are assigned to GPU A2. The positional word embedding matrix (Eevp ) is also distributed between GPU A1 and GPU A2. Matrix multiplication is then performed between the corresponding fragment of Eevp  and the respective shards of the Q, K, and V matrices. 

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEh4YA_oFjMFIunhZpSu0tguB_3t2cJF-JnLjE1ZnBW1baPpn6A2cYrPHsKeXHvkSTvtBzFI9GK0JAB9_1qDocIv2JezBIU6oQsI6_3OwqUlcL9VGUtAm5A0AB_uIg4PA5ppP4EGupPRdZ_Jezg_r-0Ji-38P5QOrKw7FqyzotJ5X4JKWWD0Cqu4AdhxDM0=w640-h362)](https://blogger.googleusercontent.com/img/a/AVvXsEh4YA_oFjMFIunhZpSu0tguB_3t2cJF-JnLjE1ZnBW1baPpn6A2cYrPHsKeXHvkSTvtBzFI9GK0JAB9_1qDocIv2JezBIU6oQsI6_3OwqUlcL9VGUtAm5A0AB_uIg4PA5ppP4EGupPRdZ_Jezg_r-0Ji-38P5QOrKw7FqyzotJ5X4JKWWD0Cqu4AdhxDM0)

  

**Figure 8-17:** _Shared Query (Q), Key (K), and Value (V) Matrices._

  

  

Figure 8-18 illustrates the cross-GPU communication involved in the forward pass of the Self-Attention layer when using Tensor Parallelism. In this example, both the word embedding, and positional embedding matrices fit within GPU A1. After computing the positional word embeddings for the input words, the resulting vectors are broadcasted to GPU A2.

  

Since we are using Tensor Parallelism, the Query (Q), Key (K), and Value (V) matrices are partitioned across GPU A1 and GPU A2. Once each GPU has computed its assigned slices of the Q, K, and V vectors, the Q and K vectors are shared between GPUs using an All-Gather operation. This ensures that each GPU receives the missing parts of the Q and K matrices, reconstructing the complete matrices across GPUs. Only the Q and K matrices are synchronized; the V matrix remains local to each GPU.

  

The Q and K matrices are then used in the Self-Attention layer, where the first operation is a matrix multiplication between the Query vectors and Key vectors for all tokens. The process is explained in detail in Chapter 7. The resulting scores are used to compute logits, which are inputs to the SoftMax function, using scaled dot-product attention. The output of the SoftMax function is then multiplied by the local fragment of the V matrix on each GPU.

  

The SoftMax operation produces a Context Vector (Cv) for each input word, which serves as the input to the Feedforward Neural Network (FFN) layer. That said, the SoftMax in the self-attention layer is not the final prediction layer, it’s used to compute attention weights. The feedforward network processes the context vectors token representations produced by self-attention, not the predicted token. The final prediction is typically made by a separate output projection followed by a SoftMax over the vocabulary.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEj-XLTMwOXsuQeXNV2KuIu9FOvvjRsLA7i3c7rCSx58dwNLase0BC0BafCUF12tnS_FdTKjkXEIcZ5X--P9Lw0kwLcsfrwDHsL_zzCrHNRhjyW2ztPOeFRxt_oNDJiT2HR3vTZFUxZOWQ-QuEfjBAl3lOtaUfn_uv38SHykSp0RAwyuwj0XWgNyXUFag0U=w640-h364)](https://blogger.googleusercontent.com/img/a/AVvXsEj-XLTMwOXsuQeXNV2KuIu9FOvvjRsLA7i3c7rCSx58dwNLase0BC0BafCUF12tnS_FdTKjkXEIcZ5X--P9Lw0kwLcsfrwDHsL_zzCrHNRhjyW2ztPOeFRxt_oNDJiT2HR3vTZFUxZOWQ-QuEfjBAl3lOtaUfn_uv38SHykSp0RAwyuwj0XWgNyXUFag0U)

**Figure 8-18:** _Tensor Parallelism in Self-Attention Layer._

  

  

#### Feedforward Layer

  

Figure 8-19 illustrates a Feedforward layer in the decoder module of a transformer. The feedforward network consists of two hidden layers and an output layer. In addition to Tensor Parallelism, we also employ Model Parallelism with Pipeline Parallelism.

  

The first hidden layer is split between GPU A1 and GPU B1, both located in the same server. The weight matrices for neurons 1–3 reside in GPU A1, while the weight matrices for neurons 4–6 are in GPU B1. The inter-GPU communication between GPU A1 and GPU B1 occurs over NVLinks, which I refer to as the High-speed Domain (HsD).

  

The second hidden layer is distributed across GPU A2 and GPU B2 within the same server. GPU A2 holds the weight matrices for neurons 1–2, while GPU B2 contains the weight matrices for neurons 3–4. The inter-GPU connection between GPU A2 and GPU B2 also utilizes NVLinks.

  

The output layer is divided between GPU A3 and GPU B3, both residing in the same server. The weight matrix for neuron 1 is stored in GPU A3, while the weight matrix for neuron 2 is in GPU B3. Inter-GPU communication occurs over NVLinks.

  

Additionally, GPU A1, GPU A2, and GPU A3 are interconnected via Rail Switch-1 across the Backend Network. Similarly, GPU B1, GPU B2, and GPU B3 are connected via Rail Switch-2 across the Backend Network.

  

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgqWeamqv7fjZHGhuWTKWQTfJ5Q0XLumXoTMh4s7Dqk9k2TTEIGrUSzU6YQCxFNQMLTgGohpLkATCsGF02grUI1fCpcnvDTQ5PCVsxnOfigd831pIpY8cMQdTeJ_gsEunI1xe7h9kta-Cse0OWezWi-yBKFPcSbLQ3IXLJWWK26KqT1Lgxl0gjp7ydVKHA=w640-h358)](https://blogger.googleusercontent.com/img/a/AVvXsEgqWeamqv7fjZHGhuWTKWQTfJ5Q0XLumXoTMh4s7Dqk9k2TTEIGrUSzU6YQCxFNQMLTgGohpLkATCsGF02grUI1fCpcnvDTQ5PCVsxnOfigd831pIpY8cMQdTeJ_gsEunI1xe7h9kta-Cse0OWezWi-yBKFPcSbLQ3IXLJWWK26KqT1Lgxl0gjp7ydVKHA)

**Figure 8-19:** _Tensor, Model and Pipeline Parallelism in Feedforward Layer._

  

  

#### Backpropagation

  

##### Forward pass

  

First Hidden Layer (H1): The input to H1, the output of the Self-Attention block after the Add & Norm step (context vectors), is shared with GPU A1 and GPU B1. Each GPU then performs its local matrix multiplication. After these local computations are complete, the partial outputs are synchronized between GPU A1 and GPU B1 using an All-Gather operation. This synchronization ensures that the complete H1 output (ynA1+B1) is calculated before it is passed to the next stage. Because GPU A1 and GPU B1 reside on the same server, the communication occurs over a high-speed domain via NVLink.

  

In the context of pipeline parallelism, H1 constitutes one pipeline stage. Once its context vector-based output is fully assembled, it is sent to the GPUs responsible for the next layer. Specifically, GPU A1 and GPU B1 first pass the output computed from the first context vector (C1), and then the GPUs process the next context vector. This communication occurs over the backend network. GPU A1, GPU A2, and GPU A3 are all connected to the same rail switch, so the RDMA packets traverse only one switch. The same design applies to GPU B1, GPU B2, and GPU B3. If communication between GPUs connected to different rail switches is required, the rail switches must be interconnected via spine switches. Alternatively, the RDMA packets may be sent over the high-speed domain to a GPU on the same rail as the destination GPU.

  

Second Hidden Layer (H2): The complete output from H1 (obtained after synchronization in the previous stage) is pipelined to GPUs A2 and B2. Each of these GPUs performs its own local matrix multiplication. As before, after the local computations, the partial outputs from GPU A2 and GPU B2 are synchronized via an All-Gather operation, forming the complete H2 output (ynA2+B2).

  

The synchronization and forwarding between hidden layer 2 and output layer, and within an output layer follow the same model as in the previous hidden layers.

  

This hybrid approach, using tensor parallelism within each stage and pipeline parallelism across stages, helps balance the computational load and memory usage across the six GPUs while minimizing idle time.

  

Although the focus of this section is on tensor parallelism, pipeline parallelism is also discussed because large language models (LLMs) can process multiple sentences from their vocabulary simultaneously during the training process.

  

On the other hand, during the inference when answering to our questions, LLMs use autoregressive next-word prediction. In this process, the final SoftMax layer of the Transformer calculates the probabilities over the vocabulary to predict the next token. This predicted token is then converted into a word and mapped to a new token. The lookup process assigns the token a positional embedding vector, which is used to compute the Query, Key, and Value vectors that feed into the Transformer's self-attention layer. Consequently, pipeline parallelism is not required during the inference phase.

  

##### Backward pass

  

The error propagates backward from the Feedforward Neural Network (FFNN) layer to the Self-Attention layer. The backpropagation process in a Transformer follows a sequential order, meaning the error from the output propagates first to the FFNN layer, and from there, it continues backward to the Self-Attention mechanism.

  

The process begins at the output layer, where the error is computed using the SoftMax function and cross-entropy loss. This error is then backpropagated through the FFNN layer, where gradients for the weight matrices are computed. Since the FFNN weights are split across multiple GPUs in Tensor Parallelism, each GPU computes its local gradient. An All-Reduce operation is then performed to synchronize these gradients across GPUs, ensuring that all GPUs have the correct weight updates before proceeding.

  

Once the gradients for the FFNN weights are synchronized, the error propagates back to the Self-Attention layer. Here, gradients for the Query (Q), Key (K), and Value (V) matrices are computed. Since these matrices were split across GPUs during the forward pass, the missing Q and K fragments must be gathered before calculating gradients. An All-Gather operation is used to collect Q and K values across GPUs. Once each GPU has a complete Q and K matrix, it computes the required gradients locally. After the local gradient computation, an All-Reduce operation is performed to ensure all GPUs have the synchronized gradients before updating the weights.

  

After both layers complete their gradient computations and synchronizations, the optimizer updates the weights, and the next iteration begins. The key communication phases include All-Gather for assembling required Q and K values before gradient computation and All-Reduce for synchronizing gradients before weight updates.


**References:**
* https://nwktimes.blogspot.com/2025/03/parallelism-strategies-in-deep-learning.html
* https://nwktimes.blogspot.com/2025/03/model-parallelism-with-pipeline.html
* https://nwktimes.blogspot.com/2025/03/tensor-parallelism.html
