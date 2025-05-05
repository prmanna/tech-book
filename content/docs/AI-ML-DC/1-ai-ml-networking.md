---
title: "1. AI/ML Networking"
bookCollapseSection: true
weight: 10
---

# Intro
## AI/ML Networking Part I: RDMA Basics
TBD

## AI/ML Networking: Part-II: Introduction of Deep Neural Networks
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

### Input Layer: 
The first layer doesn’t have neurons, instead the input data parameters X1, X2, and X3 are in this layer, from where they are fed to first hidden layer. 

### Hidden Layer: 
The neurons in the hidden layer calculate a weighted sum of the input data, which is then passed through an activation function. In our example, we are using the Rectified Linear Unit (ReLU) activation function. These calculations produce activation values for neurons. The activation value is modified input data value received from the input layer and published to upper layer.

### Output Layer: 
Neurons in this layer calculate the weighted sum in the same manner as neurons in the hidden layer, but the result of the activation function is the final output.

The process described above is known as the Forwarding pass operation. Once the forward pass process is completed, the result is passed through a loss function, where the received value is compared to the expected value. The difference between these two values triggers the backpropagation process. The Loss calculation is the initial phase of Backpropagation process. During backpropagation, the network fine-tunes the weight values , neuron by neuron, from the output layer through the hidden layers. The neurons in the input layer do not participate in the backpropagation process because they do not have weight values to be adjusted.

After the backpropagation process, a new iteration of the forward pass begins from the first hidden layer. This loop continues until the received and expected values are close enough to expected value, indicating that the training is complete.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/2-1.jpg)

**Figure 2-1:** _Deep Neural Network Basic Structure and Operations._

### Forwarding Pass 
Next, let's examine the operation of a Neural Network in more detail. Figure 2-2 illustrates a simple, three-layer Feedforward Neural Network (FNN) data model. The input layer has two neurons, H1 and H2, each receiving one input data value: a value of one (1) is fed to neuron H1 by input neuron X1, and a value of zero (0) is fed to neuron H2 by input neuron X2. The neurons in the input layer do not calculate a weighted sum or an activation value but instead pass the data to the next layer, which is the first hidden layer.

The hidden layer in our example consists of two neurons. These neurons use the ReLU activation function to calculate the activation value. During the initialization phase, the weight values for these neurons are assigned using the He Initialization method, which is often used with the ReLU function. The He Initialization method calculates the variance as 2/_n_ where _n_ is the number of neurons in the previous layer. In this example, with two input neurons, this gives a variance of  1 (=2/2). The weights are then drawn from a normal distribution ~_N(0,√variance),_ which in this case is  ~N(0,1). Basically, this means that the randomly generated weight values are centered around zero with a standard deviation of one.

In Figure 2-2, the weight value for neuron H3 in the hidden layer is 0.5 for both input sources X1 (input data 1) and X2 (input data 0). Similarly, for the hidden layer neuron H4, the weight value is 1 for both input sources X1 (input data 1) and X2 (input data 0). Neurons in the hidden and output layers also have a bias variable. If the input to a neuron is zero, the output would also be zero if there were no bias. The bias ensures that a neuron can still produce a meaningful output even when the input is zero (i.e., the neuron is inactive). Neurons H3 and O5 have a bias value of 0.5, while neuron H4 has a bias value of 0 (I am using zero for simplify the calculation). 

Let’s start the forward pass process from neuron H3 in the hidden layer. First, we calculate the weighted sum using the formula below, where Z3 represents the weighted sum of input. Here, Xn is the actual input data value received from the input layer’s neuron, and Wn  is the weight associated with that particular input neuron.

The weighted sum calculation (Z3) for neuron H3:
```
Z3 = (X1 ⋅ W31) + (X2 ⋅ W32) + b3

Given:
Z3 = (1 ⋅ 0.5) + (0 ⋅ 0.5) + 0
Z3 = 0.5 + 0 + 0
Z3 = 0.5
```

To get the activation value a3 (shown as H3=0.5 in figure), we apply the ReLU function. The ReLU function outputs zero (0) if the calculated weighted sum Z is less than or equal to zero; otherwise, it outputs the value of the weighted sum Z.
  
The activation value a3 for H3 is:
  
> _ReLU (Z3) = ReLU (0.5) = 0.5_

The weighted sum calculation for neuron H4:

```
Z4 = (X1 ⋅ W41) + (X2 ⋅ W42) + b4

Given:
Z4 = (1 ⋅ 1) + (0 ⋅1) + 0.5
Z4 = 1 + 0 + 0.5
Z4 = 1.5
```

The activation value using ReLU for Z4 is:

> _ReLU (Z4) = ReLU (1.5) = 1.5_

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/2-2.jpg)

**Figure 2-2:** _Forwarding Pass on Hidden Layer._

After neurons H3 and H4 publish their activation values to neuron O5 in the output layer, O5 calculates the weighted sum Z5 for inputs with weights W53=1and W54=1. Using Z5, it calculates the output using the ReLU function. The difference between the received output value (Yr) and the expected value (Ye) triggers a backpropagation process. In our example, Yr−Ye=0.5.

### Backpropagation process

The loss function measures the difference between the predicted output and the actual expected output. The loss function value indicates how well the neural network is performing. A high loss value means the network's predictions are far from the actual values, while a low loss value means the predictions are close.

After calculating the loss, backpropagation is initiated to minimize this loss. Backpropagation involves calculating the gradient of the loss function with respect to each weight and bias in the network. This step is crucial for adjusting the weights and biases to reduce the loss in subsequent forwarding pass iterations.

Loss function is calculated using the formula below:

```
Loss (L) = (H3 x W53 + H4 x W54 + b5 – Ye)2
Given:
L = (0.5 x 1 + 1.5 x 1 + 0.5 - 2)2
L = (0.5 + 1.5 + 0.5 - 2)2
L = 0.52
L= 0.25
```

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/2-3.jpg)

**Figure 2-3:** _Forwarding Pass on Output Layer._

The result of the loss function is then fed into the gradient calculation process, where we compute the gradient of the loss function with respect to each weight and bias in the network. The gradient calculation result is then used to fine-tune the old weight values. The Eta hyper-parameter _η_ (the learning rate) controls the step size during weight updates in the backpropagation process, balancing the speed of convergence with the stability of training. In our example, we are using a learning rate of 1/100 = 0.01. The term hyper-parameters refers to parameters that affect the final result.

First, we compute the partial derivative of the loss function (gradient calculation) with respect to the old weight values. The following example shows the gradient calculation for weight W53. The same computation applies to W54  and b3.

**Gradient Calculation:**

```
∂L = 2W53 x (Yr – Ye)

∂W53
Given
= 2 x 0.5 x (2.5 - 2)
= 1 x 0.5
= 0.5
```

**New weight value calculation.**

```
 W53 (new) = W53(old) – η x ∂L/∂W53
 
 Given:
 W53 (new) = 1–0.01 x 0.5
 W53 (new) = 0.995
```

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/2-4.jpg)

**Figure 2-4:** _Backpropagation - Gradient Calculation and New Weight Value Computation._

Figure 2-5 shows the formulas for calculating the new bias b3. The process is the same than what was used with updating the weight values.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/2-5.jpg)

**Figure 2-5:** _Backpropagation - Gradient Calculation and New Bias Computation._

After updating the weights and biases, the backpropagation process moves to the hidden layer. Gradient computation in the hidden layer is more complex because the loss function only includes weights from the output layer as you can see from the Loss function formula below:

Loss (L) = (H3 x W53 + H4 x W54 \+ b5 – Ye)2

The formula for computing the weights and biases for neurons in the hidden layers uses the chain rule. The mathematical formula shown below, but the actual computation is beyond the scope of this chapter.

∂L   =    ∂L  x  ∂H3    

∂W31   ∂H3    ∂W31    

After the backpropagation process is completed, the next iteration of the forward pass starts. This loop continues until the received result is close enough to the expected result.

If the size of the input data exceeds the GPU’s memory capacity or if the computing power of one GPU is insufficient for the data model, we need to decide on a parallelization strategy. This strategy defines how the training workload is distributed across several GPUs. Parallelization impacts network load if we need more GPUs than are available on one server. Dividing the workload among GPUs within a single GPU-server or between multiple GPU-servers triggers synchronization of calculated gradients between GPUs. When the gradient is calculated, the GPUs synchronize the results and compute the average gradient, which is then used to update the weight values.

The upcoming chapter introduces pipeline parallelization and synchronization processes in detail. We will also discuss why lossless connection is required for AI/ML.

## AI/ML Networking: Part-IV: Convolutional Neural Network (CNN) Introduction

Feed-forward Neural Networks are suitable for simple tasks like basic time series prediction without long-term relationships. However, FNNs is not a one-size-fits-all solution. For instance, digital image training process uses pixel values of image as input data. Consider training a model to recognize a high resolution (600 dpi), 3.937 x 3.937 inches digital RGB (red, green, blue) image. The number of input parameters can be calculated as follows:

> Width: 3.937 in x 600 ≈ 2362 pixels  
> Height: 3.937 in x 600 ≈ 2362 pixels  
> Pixels in image: 2362 x 2362 = 5,579,044 pixels  
> RGB (3 channels): 5,579,044 pxls x 3 channels = 16 737 132  
> Total input parameters: 16 737 132  
> Memory consumption: ≈ 16 MB

FNNs are not ideal for digital image training. If we use FNN for training in our example, we fed 16,737,132 input parameters to the first hidden layer, each having unique weight. For image training, there might be thousands of images, handling millions of parameters demands significant computation cycles and is a memory-intensive process. Besides, FNNs treat each pixel as an independent unit. Therefore, FNN algorithm does not understand dependencies between pixels and cannot recognize the same image if it shifts within the frame. Besides, FNN does not detect edges and other crucial details. 

A better model for training digital images is Convolutional Neural Networks (CNNs). Unlike in FFN neural networks where each neuron has a unique set of weights, CNNs use the same set of weights (Kernel/Filter) across different regions of the image, which reduces the number of parameters. Besides, CNN algorithm understands the pixel dependencies and can recognize patterns and objects regardless of their position in the image. 

The input data processing in CNNs is hierarchical. The first layer, convolutional layers, focuses on low-level features such as textures and edges. The second layer, pooling layer, captures higher-level features like shapes and objects. These two layers significantly reduce the input data parameters before they are fed into the neurons in the first hidden layer, the fully connected layer, where each neuron has unique weights (like FNNs).

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-0.jpg)

### Convolution Layer

The convolution process uses a shared kernel (also known as filters), which functions similarly to a neuron in a feed-forward neural network. The kernel's coverage area, 3x3 in our example, defines how many pixels of an input image is covered at a given stride. The kernel assigns a unique weight (w) to each covered pixel (x) in a one-to-one mapping fashion and calculates the weighted sum (z) from the input data. For instance, in figure 3-1 the value of the pixel W1 is multiplied with the weight value W1 (X1W1), and pixel X2 is multiplies with weight value W2 (X2W2) and so on. Then the results are summed, which returns a weighted sum Z.  The result Z is then passed through the ReLU function, which defines the value of the new pixel (P1). This new pixel is then placed into a new image matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-1.jpg)

**Figure 3-1:** _CNN Overview – Convolution, Initial State (Stride 0)._

After calculating the new pixel value for the initial coverage area with stride zero, the kernel shifts to the right by the number of steps defined in the kernel's stride value. In our example, the stride is set to one, and the kernel moves one step to the right, covering pixels shown in Figure 3-2. Then the weighted sum is (z) calculated and run through the ReLU function, and the second pixel is added to the new matrix. Since there are no more pixels to the right, the kernel moves down by one step and returns to the first column (Figure 3-3).

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-2.jpg)

**Figure 3-2:** _CNN Overview – Convolution, Second State (Stride 1)._

Figures 3-3 and 3-4 shows the last two steps of the convolution. Notice that we have used the same weight values in each iteration. In the initial state weight w1 was associated with the first pixel (X1W1), and in the second phase with the second pixel (X2W1) and so on. The new image matrix produced by the convolutional process have decreased the 75% from the original digital image. 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-3.jpg)

**Figure 3-3:** CNN Overview – Convolution, Third State (Stride 2).
  
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-4.jpg)

**Figure 3-4:** _CNN Overview – Convolution, Fourth State (Stride 3)._

If we don't want to decrease the size of the new matrix, we must use padding. Padding adds pixels to the edges of the image. For example, a padding value of one (1) adds one pixel to the image edges, providing a new matrix that is the same size as the original image.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-5.jpg)

**Figure 3-5:** _CNN Overview – Padding._

Figure 3-6 illustrates the progression of the convolution layer from the initial state (which I refer to as stride 0) to the final phase, stride 3. The kernel covers a 3x3 pixel area in each phase and moves with a stride of 1. I use the notation SnXn to denote the stride and the specific pixel. For example, in the initial state, the first pixel covered by the kernel is labeled as S0X1, and the last pixel is labeled as S0X11.

When the kernel shifts to the left, covering the next region, the first pixel is marked as S1X2 and the last as S1X12. The same notation is applied to the weighted sum calculation. The weighted value for the first pixel is represented as (S0X1) · W1 = Z1 and for the last one as (S0X11) · W9 = Z9. The weighted input for all pixel values is then summed, and a bias is added to obtain the weighted sum for the given stride.

In the initial state, this calculation results in = Z0, which is passed through the ReLU function. The output of this function provides the value for the first pixel in the new image matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-6.jpg)

**Figure 3-6:** _CNN Overview – Convolution Summary._

### Convolution Layer Example
  
In Figure 3-7, we have a simple 12x12 = 144 pixels grayscale image representing the letter "H." In the image, the white pixels have a binary value of 255, the gray pixels have a binary value of 87, and the darkest pixels have a binary value of 2. Our kernel size is 4x4, covering 16 pixels in each stride. Because the image is grayscale, we only have one channel. The kernel uses the ReLU activation function to determine the value for the new pixel. 

Initially, at stride 0, the kernel is placed over the first region in the image. The kernel has a unique weight value for each pixel it covers, and it calculates the weighted sum for all 16 pixels. The value of the first pixel (X1 = 87) is multiplied by its associated kernel weight (W1 = 0.12), which gives us a new value of 10.4. This computation runs over all 16 pixels covered by the kernel (results shown in Figure 3-7). The new values are then summed, and a bias is added, giving a weighted sum of Z0 = 91.4. Because the value of Z0 is positive, the ReLU function returns an activation value of 91.4 (if Z > 0, Z = Z; otherwise, Z = 0). The activation value of 91.4 becomes the value of our new pixel in the new image matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-7.jpg)

**Figure 3-7:** _Convolution Layer Operation Example – Stride 0 (Initial State)._

Next, the kernel shifts one step to the right (Stride 1) and multiplies the pixel values by the associated weights. The changing parameters are the values of the pixels, while the kernel weight values remain the same. After the multiplication process is done and the weighted sum is calculated, the result is run through the ReLU function. At this stride, the result of the weighted sum (Z1) is negative, so the ReLU function returns zero (0). This value is then added to the matrix. At this phase, we have two new pixels in the matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-8.jpg)

**Figure 3-8:** _Convolution Layer Operation Example – Stride 1._

The next four figures 3-9, 3-10, and 3-11 illustrates how the kernel is shifted over the input digital image and producing a new 9x9 image matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-9.jpg)

**Figure 3-9:** _Convolution Layer Operation Example – Stride 2._

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-10.jpg)

**Figure 3-10:** _Convolution Layer Operation Example – Stride 8._

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-11.jpg)

**Figure 3-11:** _Convolution Layer Operation Example – Stride 10._

Figure 3-12 illustrates the completed convolutional layer computation. At this stage, the number of pixels in the original input image has decreased from 144 to 81, representing a reduction of 56.25%.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-12.jpg)

**Figure 3-12:** _Convolution Layer Operation Example – The Last Stride._ 

### Pooling Layer
  
After the original image is processed by the convolution layer, the resulting output is used as input data for the next layer, the pooling layer. The pooling layer performs a simpler operation than the convolution layer. Like the convolution layer, the pooling layer uses a kernel to generate new values. However, instead of applying a convolution operation, the pooling layer selects the highest value within the kernel (if MaxPooling is applied) or computes the average of the values covered by the kernel (Average Pooling).

In this example, we use MaxPooling with a kernel size of 2x2 and a stride of 2. The first pixel is selected from values 91, 0, 112, and 12, corresponding to pixels in positions 1, 2, 10, and 11, respectively. Since the pixel at position 10 has the highest value (112), it is selected for the new matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-13.jpg)

**Figure 3-13:** _Pooling Layer – Stride 0 (Initial Phase)._

After selecting the highest value from the initial phase, the kernel moves to the next region, covering the values 252, 153, 212, and 52. The highest value, 252, is then placed into the new matrix. 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-14.jpg)

**Figure 3-14:** _Pooling Layer – Stride 1._

Figure 3-15 illustrates how MaxPooling progresses to the third region, covering the values 141, 76, 82, and 35. The highest value, 141, is then placed into the matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-15.jpg)

**Figure 3-15:** _Pooling Layer – Stride 2._

Figure 3-16 describes how the original 12x12 (144 pixels) image is first processed by the convolution layer, reducing it to a 9x9 (81 pixels) matrix, and then by the pooling layer, further reducing it to a 5x5 (25 pixels) matrix.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-16.jpg)

**Figure 3-16:** _CNN Convolution and Pooling Layer Results._

As the final step, the matrix generated by the pooling layer is flattened and used as input for the neurons in the fully connected layer. This means we have 25 input neurons, each with a unique weight assigned to every input parameter. The neurons in the input layer calculate the weighted sum, which neurons then use to determine the activation value. This activation value serves as the input data for the output layer. The neurons in the output layer produce the result based on their calculations, with the output H proposed by three output neurons.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/3-17.jpg)

**Figure 3-17:** _The Model of Convolution Neural Network (CNN)._

**References:**
* https://nwktimes.blogspot.com/2024/06/aiml-networking-part-i-rdma-basics.html
* https://nwktimes.blogspot.com/2024/07/aiml-networking-part-ii-introduction-of.html
* https://nwktimes.blogspot.com/2024/07/aiml-networking-part-iii-basics-of.html
* https://nwktimes.blogspot.com/2024/08/aiml-networking-part-iv-convolutional.html
