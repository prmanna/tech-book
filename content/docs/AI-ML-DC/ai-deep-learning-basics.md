---
title: "Deep Learning Basics | Artificial Neuron"
bookCollapseSection: true
weight: 10
---

#### Content

* Introduction 
* Artificial Neuron   
  * Weighted Sum for Pre-Activation Value   
  * ReLU Activation Function for Post-Activation   
  * Bias Term  
  * S-Shaped Functions – TANH and SIGMOID
* Network Impact  
* Summary  
  
## Introduction

Artificial Intelligence (AI) is a broad term for solutions that aim to mimic the functions of the human brain. Machine Learning (ML), in turn, is a subset of AI, suitable for tasks like simple pattern recognition and prediction. Deep Learning (DL), the focus of this section, is a subset of ML that leverages algorithms to extract meaningful patterns from data. Unlike ML, DL does not necessarily require human intervention, such as providing structured, labeled datasets (e.g., 1,000 bird images labeled as “bird” and 1,000 cat images labeled as “cat”). 

DL utilizes layered, hierarchical Deep Neural Networks (DNNs), where hidden and output layers consist of computational units, artificial neurons, which individually process input data. The nodes in the input layer pass the input data to the first hidden layer without performing any computations, which is why they are not considered neurons or computational units. Each neuron calculates a pre-activation value (z) based on the input received from the previous layer and then applies an activation function to this value, producing a post-activation output (ŷ) value. There are various DNN models, such as Feed-Forward Neural Networks (FNN), Convolutional Neural Networks (CNN), and Recurrent Neural Networks (RNN), each designed for different use cases. For example, FNNs are suitable for simple, structured tasks like handwritten digit recognition using the MNIST dataset \[1\], CNNs are effective for larger image recognition tasks such as with the CIFAR-10 dataset \[2\], and RNNs are commonly used for time-series forecasting, like predicting future sales based on historical sales data. 

To provide accurate predictions based on input data, neural networks are trained using labeled datasets. The MNIST (Modified National Institute of Standards and Technology) dataset \[1\] contains 60,000 training and 10,000 test images of handwritten digits (grayscale, 28x28 pixels). The CIFAR-10 \[2\] dataset consists of 60,000 color images (32x32 pixels), with 50,000 training images and 10,000 test images, divided into 10 classes. The CIFAR-100 dataset \[3\], as the name implies, has 100 image classes, with each class containing 600 images (500 training and 100 test images per class). Once the test results reach the desired level, the neural network can be deployed to production.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-1.jpg)

**Figure 1-1:** _Deep Learning Introduction._

## Artificial Neuron

As we saw in the previous section, DNNs leverage a hierarchical, layered, role-based (input layer, n hidden layers, and output layer) network model, where each layer consists of artificial neurons. Neurons in different layers may be fully or partially connected to neurons in the other layers, depending on the DNN’s network model. However, neurons within the same layer are not connected.

The logical structure and functionality of an artificial neuron aim to emulate those of a biological neuron. A biological neuron transmits electrical signals to its peer neuron via the output axon to the axon terminal, which connects to the dendrites of the peer neuron at a connection point called a synapse. A biological neuron may receive signals from multiple dendrites, and if the signal is strong enough, it activates the neuron, causing the cell body to trigger a signal through its output axon, which connects to another neuron, and the process continues.

An artificial neuron, as a computational unit, calculates the weighted sum (z = pre-activation value) of the input (x) received from the previous layer, adds a bias value (b), and applies an activation function to produce the output (ŷ = post-activation value). 

The weight value for the input can be loosely compared to a synapse since it represents a connection, input values are assigned with weight. The weights-to-neuron association, in turn, can be seen as analogous to dendrites. The computational processes (weighted sum of inputs, bias addition, and activation functions) represent the cell body, while the connections to other neurons can be compared to output axons. In Figure 1-2, bn refers to a biological neuron. From now on, "neuron" will refer to an artificial neuron.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-2.jpg)

**Figure 1-2:** _Construct of an Artificial Neuron._

### Weighted Sum for Pre-Activation Value

The lower part of Figure 1-2 depicts the mathematical formulas of a neuron. The _pre-activation_ value z is the weighted sum of the inputs. Although the bias is not part of the input data, a neuron treats it as an input variable when calculating the weighted sum. Each input x has a corresponding weight w. The calculation process is straightforward: each input value is multiplied by its corresponding weight, and the results are summed to obtain the weighted sum z.

The capital Greek letter sigma ∑ in the formula indicates that we are summing a series of terms, which, in this case, are the input values multiplied by their respective weights. The letter n specifies how many terms are being summed (four pairs of inputs and weights), while the letter iii denotes the starting point of the summation (bias b0 and weight w0). The equation for the weighted sum is:

z = b0w0 + x1w1 + x2w2 + x3w3. 

### ReLU Activation Function for Post-Activation

Next, the process applies an activation function, ReLU (Rectified Linear Unit) \[4\] in our example, to the weighted sum z obtain the post-activation value ŷ. The output of the ReLU function is z if z is greater than zero (0); otherwise, the result is zero (0). This can be written as: 

ŷ = MAX (0, z) 

which selects the larger value between 0 and variable z.  The figure 1-3 depicts the ReLU activation function.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-3.jpg)

**Figure 1-3:** _Construct of an Artificial Neuron._

Based on the figure 1-3 we can use the mathematical definition below for ReLU:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-3-kaava.jpg)

### Bias Term

In the example calculation above, imagine that all input values are zero. Without a bias term, the activation value will be zero, regardless of how large the weight parameters are. Therefore, the bias term allows the neuron to produce non-zero outputs, even when all input values are zero.

### S-Shaped Functions – TANH and SIGMOID

The ReLU function is a non-linear activation function. Naturally, there are other activation functions as well. The Hyperbolic Tangent (tanh) \[5\] and the logistic Sigmoid \[6\] functions are examples of S-shaped functions that are symmetric around zero. Figure 1-4 illustrates that as the positive z value increases, the tanh function approaches one (1), while as the negative z value decreases, it approaches -1. Thus, the range of the tanh function is from -1 to 1. Similarly, the sigmoid function's S-curve is also symmetric around zero, but its range is from 0 to 1.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-5.jpg)

**Figure 1-4:** _Tanh and Sigmoid functions._

Here are examples of both functions using the same pre-activation value of 3.02, as in the ReLU activation function example.

Note, the ⅇ represents **Euler’s Number ⅇ≈ 2.718.** The symbol σ represents the sigmoid function.

The formula for tanh function is:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-4-tanh-1.jpg)

The tanh function for z = 3,02

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-4-tanh-2.jpg)

The formula for sigmoid function is:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-4-sigmoid-1.jpg)

The sigmoid function for z = 3,02

The symbol σ represents sigmoid function.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/1-4-sigmoid-2.jpg)

For z=3.02, both the sigmoid and tanh functions return values close to 1, but the tanh function is slightly higher, approaching its maximum of 1 more quickly than the sigmoid.

Which activation function should we use? The ReLU function has largely replaced the tanh and sigmoid functions due to its simplicity, which reduces the required computation cycles. However, if tanh and sigmoid are used, tanh is typically applied in the hidden layers, while sigmoid is used in the output layer.

## Backpropagation Algorithm: Introduction

This chapter introduces the training model of a neural network based on the Backpropagation algorithm. The goal is to provide a clear and solid understanding of the process without delving deeply into the mathematical formulas, while still explaining the fundamental operations of the involved functions. The chapter also briefly explains why, and in which phases the training job generates traffic to the network, and why lossless packet transport is required. The Backpropagation algorithm is composed of two phases: the Forward pass (computation phase) and the Backward pass (adjustment and communication phase).

In the Forward pass, neurons in the first hidden layer calculate the weighted sum of input parameters received from the input layer, which is then passed to the neuron's activation function. Note that neurons in the input layer are not computational units; they simply pass the input variables to the connected neurons in the first hidden layer. The output from the activation function of a neuron is then used as input for the connected neurons in the next layer, whether it is another hidden layer or the output layer. The result of the activation function in the output layer represents the model's prediction, which is compared to the expected value (ground truth) using the error function. The output of the error function indicates the accuracy of the current training iteration. If the result is sufficiently close to the expected value, the training is complete. Otherwise, it triggers the Backward pass process.

In the Backward pass process, the Backpropagation algorithm first calculates the derivative of the error function. This derivative is then used to compute the error term for each neuron in the model. Neurons use their calculated error terms to determine how much and in which direction the current weight values must be fine-tuned. Depending on the model and the parallelization strategy, GPUs in multi-GPU clusters synchronize information during the Backpropagation process. This process affects network utilization.

Our feedforward neural network, shown in Figure 2-1, has one hidden layer and one output layer. If we wanted our example to be a deep neural network, we would need to add additional layers, as the definition of "deep" requires two or more hidden layers. For simplicity, the input layer is not shown in the figure.

We have three input parameters connected to neuron-a in the hidden layer as follows:

* Input X1 = 0.2 > neuron-a via weight Wa1 = 0.1
* Input X2 = 0.1 > neuron-a via weight Wa2 = 0.2
* Input X3 = 0.4 > neuron-a via weight Wa3 = 0.3
* Bias ba0 = 1.0 > neuron-a via weight Wa0 = 0.6

The bias term helps ensure that the neuron is active, meaning its output value is not zero.

The input parameters are treated as constant values, while the weight values are variables that will be adjusted during the Backward pass if the training result does not meet expectations. The initial weight values are our best guess for achieving the desired training outcome. The result of the weighted sum calculation is passed to the activation function, which provides the input for neuron-b in the output layer. We use the ReLU (Rectified Linear Unit) activation function in both layers due to its simplicity. There are other activation functions, such as hyperbolic tangent (tanh), sigmoid, and softmax, but those are outside the scope of this chapter.

The input values and weights for neuron-b are:

* Neuron-a activation function output f(af) > neuron-b via weight Wb1
* Bias ba0 = 1.0 > neuron-b via weight Wa0 = 0.5

The output, Ŷ, from neuron-b represents our feedforward neural network's prediction. This value is used along with the expected result, y, as input for the error function. In this example, we use the Mean Squared Error (MSE) error function. As we will see, the result of the first training iteration does not match our expected value, leading us to initiate the Backward pass process.

In the first step of the Backward pass, the Backpropagation algorithm calculates the derivative of the error function (MSE’). Neurons-a and b use this result as input to compute their respective error terms by multiplying MSE’ with the result of the activation function and the weight value associated with the connection to the next neuron. Note that for neuron-b, there is no next layer—just the error function—so the weight parameter is excluded from the error term calculation of neuron-b. Next, the error term value is multiplied by an input value and learning rate, and this adjustment value is added to the current weight.

After completing the Backward pass, the Backpropagation algorithm starts a new iteration of the Forward pass, gradually improving the model's prediction until it closely matches the expected value, at which point the training is complete.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/2-1.jpg)

**Figure 2-1:** _Backpropagation Algorithm._

### Introduction 

This chapter introduces the training model of a neural network based on the Backpropagation algorithm. The goal is to provide a clear and solid understanding of the process without delving deeply into the mathematical formulas, while still explaining the fundamental operations of the involved functions. The chapter also briefly explains why, and in which phases the training job generates traffic to the network, and why lossless packet transport is required. The Backpropagation algorithm is composed of two phases: the Forward pass (computation phase) and the Backward pass (adjustment and communication phase).

In the Forward pass, neurons in the first hidden layer calculate the weighted sum of input parameters received from the input layer, which is then passed to the neuron's activation function. Note that neurons in the input layer are not computational units; they simply pass the input variables to the connected neurons in the first hidden layer. The output from the activation function of a neuron is then used as input for the connected neurons in the next layer. The result of the activation function in the output layer represents the model's prediction, which is compared to the expected value (ground truth) using the error function. The output of the error function indicates the accuracy of the current training iteration. If the result is sufficiently close to the expected value (error function close to zero), the training is complete. Otherwise, it triggers the Backward pass process.

As the first step in the backward pass, the backpropagation algorithm calculates the derivative of the error function, providing the output error (gradient) of the model. Next, the algorithm computes the error term (gradient) for the neuron(s) in the output layer by multiplying the derivative of each neuron’s activation function by the model's error term. Then, the algorithm moves to the preceding layer and calculates the error term (gradient) for its neuron(s). This error term is now calculated using the error term of the connected neuron(s) in the next layer, the derivative of each neuron’s activation function, and the value of the weight parameter associated with the connection to the next layer.

After calculating the error terms, the algorithm determines the weight adjustment values for all neurons simultaneously. This computation is based on the input values, the adjustment values, and a user-defined learning rate. Finally, the backpropagation algorithm refines all weight values by adding the adjustment values to the initial weights. Once the backward pass is complete, the backpropagation algorithm starts a new iteration of the forward pass, gradually improving the model's predictions until they closely match the expected values, at which point the training is complete.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-1.png)

**Figure 2-1:** _Backpropagation Overview._

### The First Iteration - Forward Pass

Training a model often requires multiple iterations of forward and backward passes. In the forward pass, neurons in the first hidden layer calculate the weighted sum of input values, each multiplied by its associated weight parameter. These neurons then apply an activation function to the result. Neurons in subsequent layers use the activation output from previous layers as input for their own weighted sum calculations. This process continues through all the layers until reaching the output layer, where the activation function produces the model's prediction.

After the forward pass, the backpropagation algorithm calculates the error by comparing the model's output with the expected value, providing a measure of accuracy. If the model's output is close to the expected value, training is complete. Otherwise, the backpropagation algorithm initiates the backward pass to adjust the weights and reduce the error in subsequent iterations.

### Neuron-a Forward Pass Calculations

#### Weighted Sum

In Figure 2-2, we have an imaginary training dataset with three inputs and a bias term. Input values and their respective initial weight values are listed below: 

* x1 = 0.2 , initial weight wa1 = 0.1
* x2 = 0.1, initial weight wa2 = 0.2
* x3 = 0.4 , initial weight wa3 = 0.3
* ba0 = 1.0 , initial weight wa0 = 0.6

From the model training perspective, the input values are constant, unchageable values, while weight values are variables which will be refined during the backward pass process.

The standard way to write the weighted sum formula is: 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-2.png)

Where:

* n = 3 represents the number of input values (x1, x2, x3).
* Each input xi  is multiplied by its respective weight wi, and the sum of these products is added to the bias term b.

In this case, the equation can be explicitly stated as:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-3.png)
  
Which with our parameters gives:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-4.png)

#### Activation Function

Neuron-a uses the previously calculated weighted sum as input for the activation function. We are using the ReLU function (Rectified Linear Unit), which is more popular than the hyperbolic tangent and sigmoid functions due to its simplicity and lower computational cost.

The standard way to write the ReLU function is:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-5.png)

Where:

f(a) represents the activation function.
z  is the weighted sum of inputs.

The ReLU function returns the z if z > 0. Otherwise, it returns 0 if z ≤ 0.

In our example, the weighted sum za is 0.76, so the ReLU function returns:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-6.png)

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-7.png)

**Figure 2-2:** _Activation Function for Neuron-a._

### Neuron-b Forward Pass Calculations

#### Weighted Sum

Besides the bias term value of 1.0,  Neuron-b uses the result provided by the activation function of neuron-a as an input to weighted sum calculation. Input values and their respective initial weight values are listed below: 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-8.png)
  
This gives us:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-9.png)

#### Activation Function

Just like Neuron-a, Neuron-b uses the previously calculated weighted sum as input for the activation function. Because the zb = 0.804 is greater than zero, the ReLU activation function f(b) returns:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-10.png)

Neuron-b is in the output layer, so its activation function result _y__<sub><span style="font-family: &quot;Cambria Math&quot;,serif; mso-bidi-font-family: Calibri; mso-bidi-font-size: 6.0pt; mso-bidi-theme-font: minor-latin;">b</span></sub>_ represents the prediction of the model. 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-11.png)

**Figure 2-3:** Activation Function for Neuron-b.

### Error Function

To keep things simple, we have used only one training example. However, in real-life scenarios, there will always be more than one training example. For instance, a training dataset might contain 10 images of cats and 10 images of dogs, each having 28x28 pixels. Each image represents a training example, giving us a total of 20 training examples. The purpose of the error function is to provide a single error metric for all training examples. In this case, we are using the Mean Squared Error (MSE).

We can calculate the MSE using the formula below where the expected value y is 1.0 and the model’s  prediction for the training example yb = 0.804. This gives an error metric of 0.019, which can be interpreted as an indicator of the model's accuracy.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-12.png)

The result of the error function is not sufficiently close to the desired value, which is why this result triggers the backward pass process.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-13.png)

**Figure 2-4:** _Calculating the Error Function for Training Examples._

### Backward Pass

In the forward pass, we calculate the model’s accuracy using several functions. First, Neuron-a computes the weighted sum Σ(za ) by multiplying the input values and the bias term with their respective weights. The output, za, is then passed through the activation function f(a), producing ya. Neuron-b, in turn, takes ya and the bias term to calculate the weighted sum Σ(zb ). The activation function f(b) then uses zb to compute the model’s output, yb. Finally, the error function f(e) calculates the model’s accuracy based on the output.

So, dependencies between function can be seen as:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-14.png)
  
The backpropagation algorithm combines these five functions to create a new error function, enew(x), using function composition and the chain rule. The following expression shows how the error function relates to the weight parameter w1 used by Neuron-a:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-15.png)

This can be expressed using the composition operator (∘) between functions:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-16.png)

Next, we use a method called gradient descent to gradually adjust the initial weight values, refining them to bring the model's output closer to the expected result. To do this, we compute the derivative of the composite function using the chain rule, where we take the derivatives of:

1. The error function (e) with respect to the activation function (b).
2. The activation function b with respect to the weighted sum (zb). 
3. The weighted sum (zb) with respect to the activation function (a).
4. The activation function (a) with respect to weighted sum (za(w1)). 

In Leibniz’s notation, this looks like:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-17.png)

Figure 2-5 illustrates the components of the backpropagation algorithm, along with their relationships and dependencies.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-18.png)

**Figure 2-5:** _The Backward Pass Overview._

#### Partial Derivative for Error Function – Output Error (Gradient)

As a recap, and for illustrating that the prediction of the first iteration fails, Figure 2-6 includes the computation for the error function (MSE = 0.019). 

As a first step, we calculate the partial derivative of the error function. In this case, the partial derivative describes the rate of change of the error function when the input variable yb changes. The derivative is called partial when one of its input values is held constant (i.e., not adjusted by the algorithm). In our example, the expected value y is constant input. The result of the partial derivative of the error function describes how the predicted output should change yb to minimize the model’s error.

We use the following formula for computing the derivative of the error function:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-19.png)

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-20.png)

**Figure 2-6:** _The Backward Pass – Derivative of the Error Function._

The following explanation is meant for readers interested in why there is a minus sign in front of the function.

When calculating the derivative, we use the Power Rule. The Power Rule states that if we have a function f(x) = xn , then its derivative is f’(x) = n ⋅ xn-1. In our case, this applies to the error function:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-21.png)
  
Using the Power Rule, the derivative becomes:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-22.png)

Next, we apply the chain rule by multiplying this result by the derivative of the inner function (y − yb), with respect to yb . Since y is treated as a constant (because it represents our target value, which doesn't change during optimization), the derivative of (y − yb) with respect to yb  is simply −1, as the derivative of − yb  with respect to yb  is −1, and the derivative of y (a constant) is 0.

Therefore, the final derivative of the error function with respect to yb  is:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-23.png)

#### Partial Derivative for the Activation Function

After computing the output error, we calculate the derivative of the activation function f(b) with respect to zb . Neuron b uses the ReLU activation function, which states that if the input to the function is greater than 0, the derivative is 1; otherwise, it is 0. In our case, the result of the activation function f(b)=0.804, so the derivative is 1.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-24.png)

#### Error Term for Neurons (Gradient)

The error term (Gradient) for neuron-b is calculated by multiplying the output error, the partial derivative of the error function,  by the derivative of the neuron's activation function. This means that now we propagate the model's error backward using it as a base value for finetuning the model accuracy (i.e., refining new weight values). This is why the term backward pass fits perfectly for the process.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-25.png)

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-26.png)

**Figure 2-7:** _The Backward Pass – Error Term (Gradient) for Neuron-b._

After computing the error term for Neuron-b, the backward pass moves to the preceding layer, the hidden layer, and calculates the error term for Neuron-a. The algorithm computes the derivative for the activation function f(a) = 1, as it did with the Neuron-b. Next, it multiplies the result by Neuron-b's error term (-0.196) and the connected weight parameter , wb1 =0.4. The result -0.0784 is the error term for Neuron-a.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-27.png)

**Figure 2-8:** _The Backward Pass – Error Term (Gradient) for Neuron-a._

#### Weight Adjustment Value

After computing error terms for all neurons in every layer, the algorithm simultaneously calculates the weight adjustment value for every weight. The process is simple, the error term is multiplied with an input value connected to weight and with learning rate (η). The learning rate balances convergence speed and training stability. We have set it to -0.6 for the first iteration. The learning rate is a hyper-parameter, meaning it is set by the user rather than learned by the model during training. It affects the behavior of the backpropagation algorithm by controlling the size of the weight updates. It is also possible to adjust the learning rate during training—using a higher learning rate at the start to allow faster convergence and lowering it later to avoid overshooting the optimal result. 

Weight adjustment value for weight wb1 and wa1 respectively:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-28.png)
  
**Note!** _It is not recommended to use a negative learning rate. I use it here because we get a good enough output for the second forward pass iteration._

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-29.png)

**Figure 2-9:** _The Backward Pass – Weight Adjustment Value for Neurons._

#### Refine Weights

As the last step, the backpropagation algorithm computes new values for every weight parameter in the model by simply summing the initial weight value and weight adjustment value.

New values for weight  parameters wb1 and wa1 respectively:

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-30.png)

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-31.png)

**Figure 2-10:** _The Backward Pass – Compute New Weight Values._

### The Second Iteration - Forward Pass

After updating all the weight values (wa0, wa1, wa2, and wa3 ), the backpropagation process begins the second iteration of the forward pass. As shown in Figure 2-11, the model output yb = 0.9982 is very close to the expected value y = 1.0. The new MSE = 0.0017, is much better than 0.019 computed in the first iteration.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-32.png)

**Figure 2-11:** The Second Iteration of the Forward Pass.

### Network Impact

Figure 2-12 shows a hypothetical example of Data Parallelization, where our training data set is split into two batches, A and B, which are processed by GPU-A and GPU-B, respectively. The training model is the same on both GPUs: Fully-Connected, with two hidden layers of four neurons each, and one output neuron in the output layer.

After computing a model prediction during the forward pass, the backpropagation algorithm begins the backward pass by calculating the gradient (output error) for the error function. Once computed, the gradients are synchronized between the GPUs. The algorithm then averages the gradients, and the process moves to the preceding layer. Neurons in the preceding layer calculate their gradient by multiplying the weighted sum of their connected neurons’ averaged gradients and connected weight with the local activation function’s partial derivative. These neuron-based gradients are then synchronized over connections. Before the process moves to the preceding layer, gradients are averaged. The backpropagation algorithm executes the same process through all layers. 

If packet loss occurs during the synchronization, it can ruin the entire training process, which would need to be restarted unless snapshots were taken. The cost of losing even a single packet could be enormous, especially if training has been ongoing for several days or weeks. Why is a single packet so important? If the synchronization between the gradients of two parallel neurons fails due to packet loss, the algorithm cannot compute the average, and the neurons in the preceding layer cannot calculate their gradient. Besides, if the connection, whether the synchronization happens over NVLink, InfiniBand, Ethernet (RoCE or RoCEv2), or wireless connection, causes a delay, the completeness of the training slows down. This causes GPU under-utilization which is not efficient from the business perspective.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/deep-learning-basics/image-33.png)

**Figure 2-12:** _Backward Pass – Gradient Synchronization and Averaging._  

To be conntinued...

**References:**
* https://nwktimes.blogspot.com/2024/09/ai4ne-ch1-artificial-neuron.html
* https://nwktimes.blogspot.com/2024/10/ai-for-network-engineers.html
