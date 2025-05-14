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


**References:**
* https://nwktimes.blogspot.com/2025/03/parallelism-strategies-in-deep-learning.html
* https://nwktimes.blogspot.com/2025/03/model-parallelism-with-pipeline.html

