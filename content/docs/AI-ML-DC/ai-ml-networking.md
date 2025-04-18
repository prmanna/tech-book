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

## AI/ML Networking: Part-III: Basics of Neural Networks Training Process
### Neural Network Architecture Overview

Deep Neural Networks (DNN) leverage various architectures for training, with one of the simplest and most fundamental being the Feedforward Neural Network (FNN). Figure 2-1 illustrates our simple, three-layer FNN.

#### Input Layer: 
The first layer doesn’t have neurons, instead the input data parameters X1, X2, and X3 are in this layer, from where they are fed to first hidden layer. 

#### Hidden Layer: 
The neurons in the hidden layer calculate a weighted sum of the input data, which is then passed through an activation function. In our example, we are using the Rectified Linear Unit (ReLU) activation function. These calculations produce activation values for neurons. The activation value is modified input data value received from the input layer and published to upper layer.

#### Output Layer: 
Neurons in this layer calculate the weighted sum in the same manner as neurons in the hidden layer, but the result of the activation function is the final output.

The process described above is known as the Forwarding pass operation. Once the forward pass process is completed, the result is passed through a loss function, where the received value is compared to the expected value. The difference between these two values triggers the backpropagation process. The Loss calculation is the initial phase of Backpropagation process. During backpropagation, the network fine-tunes the weight values , neuron by neuron, from the output layer through the hidden layers. The neurons in the input layer do not participate in the backpropagation process because they do not have weight values to be adjusted.

After the backpropagation process, a new iteration of the forward pass begins from the first hidden layer. This loop continues until the received and expected values are close enough to expected value, indicating that the training is complete.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgwQEpZVHZtvYz66gfK5OFGHUdixuV8lqUiGoipKqReMEuC1duCCyBt4NatgsKjI6ka3xbZlswzw7vI7CBLGWrz3VKF3Qq0eqnTjbxkLh-3J1iuK9ygTIfKDqyRqMNILGdK5Z9aDxePxmsLVjv0p-YAwqoAngEPUdb72q7eIDjgWO_pyir-zSv5Q-pWkQA/w640-h286/2-1.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgwQEpZVHZtvYz66gfK5OFGHUdixuV8lqUiGoipKqReMEuC1duCCyBt4NatgsKjI6ka3xbZlswzw7vI7CBLGWrz3VKF3Qq0eqnTjbxkLh-3J1iuK9ygTIfKDqyRqMNILGdK5Z9aDxePxmsLVjv0p-YAwqoAngEPUdb72q7eIDjgWO_pyir-zSv5Q-pWkQA/s4245/2-1.jpg)

**Figure 2-1:** _Deep Neural Network Basic Structure and Operations._

#### Forwarding Pass 
Next, let's examine the operation of a Neural Network in more detail. Figure 2-2 illustrates a simple, three-layer Feedforward Neural Network (FNN) data model. The input layer has two neurons, H1 and H2, each receiving one input data value: a value of one (1) is fed to neuron H1 by input neuron X1, and a value of zero (0) is fed to neuron H2 by input neuron X2. The neurons in the input layer do not calculate a weighted sum or an activation value but instead pass the data to the next layer, which is the first hidden layer.

The hidden layer in our example consists of two neurons. These neurons use the ReLU activation function to calculate the activation value. During the initialization phase, the weight values for these neurons are assigned using the He Initialization method, which is often used with the ReLU function. The He Initialization method calculates the variance as 2/_n_ where _n_ is the number of neurons in the previous layer. In this example, with two input neurons, this gives a variance of  1 (=2/2). The weights are then drawn from a normal distribution ~_N(0,√variance),_ which in this case is  ~N(0,1). Basically, this means that the randomly generated weight values are centered around zero with a standard deviation of one.

In Figure 2-2, the weight value for neuron H3 in the hidden layer is 0.5 for both input sources X1 (input data 1) and X2 (input data 0). Similarly, for the hidden layer neuron H4, the weight value is 1 for both input sources X1 (input data 1) and X2 (input data 0). Neurons in the hidden and output layers also have a bias variable. If the input to a neuron is zero, the output would also be zero if there were no bias. The bias ensures that a neuron can still produce a meaningful output even when the input is zero (i.e., the neuron is inactive). Neurons H3 and O5 have a bias value of 0.5, while neuron H4 has a bias value of 0 (I am using zero for simplify the calculation). 

Let’s start the forward pass process from neuron H3 in the hidden layer. First, we calculate the weighted sum using the formula below, where Z3 represents the weighted sum of input. Here, Xn is the actual input data value received from the input layer’s neuron, and Wn  is the weight associated with that particular input neuron.

The weighted sum calculation (Z3) for neuron H3:

> _Z3 = (X1 ⋅ W31) + (X2 ⋅ W32) + b3_
> 
> _Given:_
> _Z3 = (1 ⋅ 0.5) + (0 ⋅ 0.5) + 0_
> _Z3 = 0.5 + 0 + 0_
> _Z3 = 0.5_

To get the activation value a3 (shown as H3=0.5 in figure), we apply the ReLU function. The ReLU function outputs zero (0) if the calculated weighted sum Z is less than or equal to zero; otherwise, it outputs the value of the weighted sum Z.
  
The activation value a3 for H3 is:
  
> _ReLU (Z3) = ReLU (0.5) = 0.5_

The weighted sum calculation for neuron H4:

> _Z4 = (X1 ⋅ W41) + (X2 ⋅ W42) + b4_
> 
> _Given:_
> _Z4 = (1 ⋅ 1) + (0 ⋅1) + 0.5_
> _Z4 = 1 + 0 + 0.5_
> _Z4 = 1.5_

The activation value using ReLU for Z4 is:

> _ReLU (Z4) = ReLU (1.5) = 1.5_

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhZlF-uvw1lezXzt0RpaqF4-Xmi91b-4Krhzn9cmxDWswUtdmC8ziCnIH9DXngAg7wwJqRpfN3MingjK2DmPLYOG0utQ0dPeEVi7mhsCtY6PL8WWlUtQmTRv9L22cMtr6ALEYhJ3i5hSw5R1C1pEjjwe53BM5CdiOxjuSUpGRRi3iKPNqKTxUlWUuBbbmU/w640-h296/2-2.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhZlF-uvw1lezXzt0RpaqF4-Xmi91b-4Krhzn9cmxDWswUtdmC8ziCnIH9DXngAg7wwJqRpfN3MingjK2DmPLYOG0utQ0dPeEVi7mhsCtY6PL8WWlUtQmTRv9L22cMtr6ALEYhJ3i5hSw5R1C1pEjjwe53BM5CdiOxjuSUpGRRi3iKPNqKTxUlWUuBbbmU/s4377/2-2.jpg)

**Figure 2-2:** _Forwarding Pass on Hidden Layer._

After neurons H3 and H4 publish their activation values to neuron O5 in the output layer, O5 calculates the weighted sum Z5 for inputs with weights W53=1and W54=1. Using Z5, it calculates the output using the ReLU function. The difference between the received output value (Yr) and the expected value (Ye) triggers a backpropagation process. In our example, Yr−Ye=0.5.

#### Backpropagation process

The loss function measures the difference between the predicted output and the actual expected output. The loss function value indicates how well the neural network is performing. A high loss value means the network's predictions are far from the actual values, while a low loss value means the predictions are close.

After calculating the loss, backpropagation is initiated to minimize this loss. Backpropagation involves calculating the gradient of the loss function with respect to each weight and bias in the network. This step is crucial for adjusting the weights and biases to reduce the loss in subsequent forwarding pass iterations.

Loss function is calculated using the formula below:

> _Loss (L) = (H3 x W53 + H4 x W54 + b5 – Ye)2_
> _Given:_
> _L = (0.5 x 1 + 1.5 x 1 + 0.5 - 2)2_
> _L = (0.5 + 1.5 + 0.5 - 2)2_
> _L = 0.52_
> _L= 0.25_

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjdFOcP9Cn02czGrq25JgHW2iker9yg3-QcllyG360S9j_By-0mgt8JL1avnWINfoSNW6luO9o9vIGXmCYHvWciNc2g6Ioz4ewcbENU2hJMl1Be9uiu6xzxCzthyupv6MC67AiPADVgIYuzIuucpERmd31xHLi5RXcRkkD6b0Jq8PszyDbZ-Q1OWIhJ_6k/w640-h346/2-3.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjdFOcP9Cn02czGrq25JgHW2iker9yg3-QcllyG360S9j_By-0mgt8JL1avnWINfoSNW6luO9o9vIGXmCYHvWciNc2g6Ioz4ewcbENU2hJMl1Be9uiu6xzxCzthyupv6MC67AiPADVgIYuzIuucpERmd31xHLi5RXcRkkD6b0Jq8PszyDbZ-Q1OWIhJ_6k/s4377/2-3.jpg)

**Figure 2-3:** _Forwarding Pass on Output Layer._

The result of the loss function is then fed into the gradient calculation process, where we compute the gradient of the loss function with respect to each weight and bias in the network. The gradient calculation result is then used to fine-tune the old weight values. The Eta hyper-parameter _η_ (the learning rate) controls the step size during weight updates in the backpropagation process, balancing the speed of convergence with the stability of training. In our example, we are using a learning rate of 1/100 = 0.01. The term hyper-parameters refers to parameters that affect the final result.

First, we compute the partial derivative of the loss function (gradient calculation) with respect to the old weight values. The following example shows the gradient calculation for weight W53. The same computation applies to W54  and b3.

**Gradient Calculation:**

> _∂L   = 2W53 x (Yr – Ye)_
> 
> _∂W53_

>  _Given_
>  _= 2 x 0.5 x (2.5 - 2)_
>  _= 1 x 0.5_
>  _= 0.5_

**New weight value calculation.**

> _W53 (new) = W53(old) – η x ∂L/∂W53_
> 
> _Given:_
> _W53 (new) = 1–0.01 x 0.5_
> _W53 (new) = 0.995_

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhIx1tlhe-CTPD9BaZpXDhngVJBd3WpX4y7IdZRHwMC_dEIsvh09C4GdA8dfS3UUJ9RQhYnT50YXHNLja6LM6yJc8mEoDH9WdHfpZ_MjI8byVLk1376ieBDc8gDqoEsmaNhjmVK6y_IMFhRtrOSmfADXg-BFwlPb6eyOsHkjskmX8KIO1xJpJ7xAA6I8pk/w640-h320/2-4.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhIx1tlhe-CTPD9BaZpXDhngVJBd3WpX4y7IdZRHwMC_dEIsvh09C4GdA8dfS3UUJ9RQhYnT50YXHNLja6LM6yJc8mEoDH9WdHfpZ_MjI8byVLk1376ieBDc8gDqoEsmaNhjmVK6y_IMFhRtrOSmfADXg-BFwlPb6eyOsHkjskmX8KIO1xJpJ7xAA6I8pk/s4365/2-4.jpg)

**Figure 2-4:** _Backpropagation - Gradient Calculation and New Weight Value Computation._

Figure 2-5 shows the formulas for calculating the new bias b3. The process is the same than what was used with updating the weight values.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjgPJMFQHffuBguHEkzw1tyrczatiFp_sxKK7T8BROCP1CHwlH1y04qRUOXhM-3BKKQKERjkD3A1G4YdfZnT1ocptfrbdYPwOYkspb0Uj-YPCPmOQ5fb9aacAcPsKCnJScLHoAs6o9-Ak8eeW-FGKccTKRuoixEsSYAC2_FX7crUbapI0uT5SJop_jxoG8/w640-h322/2-5.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjgPJMFQHffuBguHEkzw1tyrczatiFp_sxKK7T8BROCP1CHwlH1y04qRUOXhM-3BKKQKERjkD3A1G4YdfZnT1ocptfrbdYPwOYkspb0Uj-YPCPmOQ5fb9aacAcPsKCnJScLHoAs6o9-Ak8eeW-FGKccTKRuoixEsSYAC2_FX7crUbapI0uT5SJop_jxoG8/s4362/2-5.jpg)

**Figure 2-5:** _Backpropagation - Gradient Calculation and New Bias Computation._

After updating the weights and biases, the backpropagation process moves to the hidden layer. Gradient computation in the hidden layer is more complex because the loss function only includes weights from the output layer as you can see from the Loss function formula below:

Loss (L) = (H3 x W53 + H4 x W54 \+ b5 – Ye)2

The formula for computing the weights and biases for neurons in the hidden layers uses the chain rule. The mathematical formula shown below, but the actual computation is beyond the scope of this chapter.

∂L   =    ∂L  x  ∂H3    

∂W31   ∂H3    ∂W31    

After the backpropagation process is completed, the next iteration of the forward pass starts. This loop continues until the received result is close enough to the expected result.

If the size of the input data exceeds the GPU’s memory capacity or if the computing power of one GPU is insufficient for the data model, we need to decide on a parallelization strategy. This strategy defines how the training workload is distributed across several GPUs. Parallelization impacts network load if we need more GPUs than are available on one server. Dividing the workload among GPUs within a single GPU-server or between multiple GPU-servers triggers synchronization of calculated gradients between GPUs. When the gradient is calculated, the GPUs synchronize the results and compute the average gradient, which is then used to update the weight values.

The upcoming chapter introduces pipeline parallelization and synchronization processes in detail. We will also discuss why lossless connection is required for AI/ML.

**References:**
