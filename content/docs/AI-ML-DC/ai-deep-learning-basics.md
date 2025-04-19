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
  
### Introduction

Artificial Intelligence (AI) is a broad term for solutions that aim to mimic the functions of the human brain. Machine Learning (ML), in turn, is a subset of AI, suitable for tasks like simple pattern recognition and prediction. Deep Learning (DL), the focus of this section, is a subset of ML that leverages algorithms to extract meaningful patterns from data. Unlike ML, DL does not necessarily require human intervention, such as providing structured, labeled datasets (e.g., 1,000 bird images labeled as “bird” and 1,000 cat images labeled as “cat”). 

DL utilizes layered, hierarchical Deep Neural Networks (DNNs), where hidden and output layers consist of computational units, artificial neurons, which individually process input data. The nodes in the input layer pass the input data to the first hidden layer without performing any computations, which is why they are not considered neurons or computational units. Each neuron calculates a pre-activation value (z) based on the input received from the previous layer and then applies an activation function to this value, producing a post-activation output (ŷ) value. There are various DNN models, such as Feed-Forward Neural Networks (FNN), Convolutional Neural Networks (CNN), and Recurrent Neural Networks (RNN), each designed for different use cases. For example, FNNs are suitable for simple, structured tasks like handwritten digit recognition using the MNIST dataset \[1\], CNNs are effective for larger image recognition tasks such as with the CIFAR-10 dataset \[2\], and RNNs are commonly used for time-series forecasting, like predicting future sales based on historical sales data. 

To provide accurate predictions based on input data, neural networks are trained using labeled datasets. The MNIST (Modified National Institute of Standards and Technology) dataset \[1\] contains 60,000 training and 10,000 test images of handwritten digits (grayscale, 28x28 pixels). The CIFAR-10 \[2\] dataset consists of 60,000 color images (32x32 pixels), with 50,000 training images and 10,000 test images, divided into 10 classes. The CIFAR-100 dataset \[3\], as the name implies, has 100 image classes, with each class containing 600 images (500 training and 100 test images per class). Once the test results reach the desired level, the neural network can be deployed to production.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgylzz8OQkWjjhyBpYDBNRNezdgowWbuzeSXr_I5uSf6XnWq2y2ZNXLKJtTcrFTyWsxR_TGMJE1-88DTfNcfYktTOCx7J87VxTkRYTEVBpxuFFRiIXisS1Qw9KjKqgOW8bmAT8_4lJgaRabWZh8b5e9T0fDrQIgghzFlJgwKn2Dvj8HJiDQ2pNLhRKKKlM/w640-h416/1-1.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgylzz8OQkWjjhyBpYDBNRNezdgowWbuzeSXr_I5uSf6XnWq2y2ZNXLKJtTcrFTyWsxR_TGMJE1-88DTfNcfYktTOCx7J87VxTkRYTEVBpxuFFRiIXisS1Qw9KjKqgOW8bmAT8_4lJgaRabWZh8b5e9T0fDrQIgghzFlJgwKn2Dvj8HJiDQ2pNLhRKKKlM/s4577/1-1.jpg)

**Figure 1-1:** _Deep Learning Introduction._

### Artificial Neuron

As we saw in the previous section, DNNs leverage a hierarchical, layered, role-based (input layer, n hidden layers, and output layer) network model, where each layer consists of artificial neurons. Neurons in different layers may be fully or partially connected to neurons in the other layers, depending on the DNN’s network model. However, neurons within the same layer are not connected.

The logical structure and functionality of an artificial neuron aim to emulate those of a biological neuron. A biological neuron transmits electrical signals to its peer neuron via the output axon to the axon terminal, which connects to the dendrites of the peer neuron at a connection point called a synapse. A biological neuron may receive signals from multiple dendrites, and if the signal is strong enough, it activates the neuron, causing the cell body to trigger a signal through its output axon, which connects to another neuron, and the process continues.

An artificial neuron, as a computational unit, calculates the weighted sum (z = pre-activation value) of the input (x) received from the previous layer, adds a bias value (b), and applies an activation function to produce the output (ŷ = post-activation value). 

The weight value for the input can be loosely compared to a synapse since it represents a connection, input values are assigned with weight. The weights-to-neuron association, in turn, can be seen as analogous to dendrites. The computational processes (weighted sum of inputs, bias addition, and activation functions) represent the cell body, while the connections to other neurons can be compared to output axons. In Figure 1-2, bn refers to a biological neuron. From now on, "neuron" will refer to an artificial neuron.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjOrkcskrlY_TpMwWARozhWCz6-0xDdUhr_765rrYx_1sloD7s1Kpwq_9o3S7CO9tg2TQBZ30MjNnUJBFD5BplD_iWUB-N51okGxDKZPpZYwsURDniDeQIpxpSOAngWnYaGj4jRG5rKq_UH9fEDGdvVQjOBnni-qji-uJLggmrEfpMG0FDYcAMNme4O8Pg/w640-h366/1-2.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjOrkcskrlY_TpMwWARozhWCz6-0xDdUhr_765rrYx_1sloD7s1Kpwq_9o3S7CO9tg2TQBZ30MjNnUJBFD5BplD_iWUB-N51okGxDKZPpZYwsURDniDeQIpxpSOAngWnYaGj4jRG5rKq_UH9fEDGdvVQjOBnni-qji-uJLggmrEfpMG0FDYcAMNme4O8Pg/s4158/1-2.jpg)

**Figure 1-2:** _Construct of an Artificial Neuron._

### Weighted Sum for Pre-Activation Value

The lower part of Figure 1-2 depicts the mathematical formulas of a neuron. The _pre-activation_ value z is the weighted sum of the inputs. Although the bias is not part of the input data, a neuron treats it as an input variable when calculating the weighted sum. Each input x has a corresponding weight w. The calculation process is straightforward: each input value is multiplied by its corresponding weight, and the results are summed to obtain the weighted sum z.

The capital Greek letter sigma ∑ in the formula indicates that we are summing a series of terms, which, in this case, are the input values multiplied by their respective weights. The letter n specifies how many terms are being summed (four pairs of inputs and weights), while the letter iii denotes the starting point of the summation (bias b0 and weight w0). The equation for the weighted sum is:

z = b0w0 + x1w1 + x2w2 + x3w3. 

### ReLU Activation Function for Post-Activation

Next, the process applies an activation function, ReLU (Rectified Linear Unit) \[4\] in our example, to the weighted sum z obtain the post-activation value ŷ. The output of the ReLU function is z if z is greater than zero (0); otherwise, the result is zero (0). This can be written as: 

ŷ = MAX (0, z) 

which selects the larger value between 0 and variable z.  The figure 1-3 depicts the ReLU activation function.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhuyOdyPBd4vJC9Q5d3kKRz_eGaHlbX5s0vc6mbdOwf6i2qz55wRX3IxTZ5jenuY1ZtR_UX16iEVrzFZu_W2QsWonDLEcZwQAmwt1_hEFKmmb2J54TjXB5cIMM480I4LNzy7MKNGMSd7ub4TNbeAsF1FRPEhkonxa6rxPqj0Kg_S5qoR_EnKW_-axuq9NI/w640-h494/1-3.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhuyOdyPBd4vJC9Q5d3kKRz_eGaHlbX5s0vc6mbdOwf6i2qz55wRX3IxTZ5jenuY1ZtR_UX16iEVrzFZu_W2QsWonDLEcZwQAmwt1_hEFKmmb2J54TjXB5cIMM480I4LNzy7MKNGMSd7ub4TNbeAsF1FRPEhkonxa6rxPqj0Kg_S5qoR_EnKW_-axuq9NI/s2879/1-3.jpg)

**Figure 1-3:** _Construct of an Artificial Neuron._

Based on the figure 1-3 we can use the mathematical definition below for ReLU:

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZOd_pTlu8aKviu_WlIqUx4wvywYR3pgvLDuHKOIiAX0AmzrSTopNH1nbcpSzC2frox8I0N9ZQwai1jdvLIii8HHH9EFjKUkXn_nWNPIZipIe00M1anltJ-uiFaVSg7NL3awe1ThXMh_AbxB75nwtSJ5jvEmxtPQXmimnDXPxj4M8QxKVagtSC2Hu-I_4/w640-h342/1-3-kaava.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZOd_pTlu8aKviu_WlIqUx4wvywYR3pgvLDuHKOIiAX0AmzrSTopNH1nbcpSzC2frox8I0N9ZQwai1jdvLIii8HHH9EFjKUkXn_nWNPIZipIe00M1anltJ-uiFaVSg7NL3awe1ThXMh_AbxB75nwtSJ5jvEmxtPQXmimnDXPxj4M8QxKVagtSC2Hu-I_4/s1051/1-3-kaava.jpg)

### Bias Term

In the example calculation above, imagine that all input values are zero. Without a bias term, the activation value will be zero, regardless of how large the weight parameters are. Therefore, the bias term allows the neuron to produce non-zero outputs, even when all input values are zero.

### S-Shaped Functions – TANH and SIGMOID

The ReLU function is a non-linear activation function. Naturally, there are other activation functions as well. The Hyperbolic Tangent (tanh) \[5\] and the logistic Sigmoid \[6\] functions are examples of S-shaped functions that are symmetric around zero. Figure 1-4 illustrates that as the positive z value increases, the tanh function approaches one (1), while as the negative z value decreases, it approaches -1. Thus, the range of the tanh function is from -1 to 1. Similarly, the sigmoid function's S-curve is also symmetric around zero, but its range is from 0 to 1.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjeIqQJqOcO1a5K7mi9pky0yQbyZET0nciIFl_NlnylFLsZJyDy1RAgB0nqJteGQhPmshYXS5ozCjddEvfP_0w4Q6UXivhqK1TAge7_jolCTqx059LHQs9raYAK54P6op1uOQm_jtO5yvFOl1Ms8MiEHIcExscjFlZJ-Iyzus026-6PFDhlLSocVqfIHDM/w640-h324/1-5.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjeIqQJqOcO1a5K7mi9pky0yQbyZET0nciIFl_NlnylFLsZJyDy1RAgB0nqJteGQhPmshYXS5ozCjddEvfP_0w4Q6UXivhqK1TAge7_jolCTqx059LHQs9raYAK54P6op1uOQm_jtO5yvFOl1Ms8MiEHIcExscjFlZJ-Iyzus026-6PFDhlLSocVqfIHDM/s4430/1-5.jpg)

**Figure 1-4:** _Tanh and Sigmoid functions._

Here are examples of both functions using the same pre-activation value of 3.02, as in the ReLU activation function example.

Note, the ⅇ represents **Euler’s Number ⅇ≈ 2.718.** The symbol σ represents the sigmoid function.

The formula for tanh function is:

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh4R5gSyFjTT5RmxaKhzg7p75b62HJ3MnSqlp4dcDuf_csfY2h1-i4HV_lnY77guRnt6a0Sy7xxmD-y16jkVXjNHSzCTamDN4VNqExSGoy8nDXDP8fttjk0sZwT0xj_s8LSuK1FzVbcpJgo9nIxLBYZTWmv57HjYA0q0IJLZyCLJWF4xDaotW3boxwtEtQ/w640-h68/1-4-tanh-1.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh4R5gSyFjTT5RmxaKhzg7p75b62HJ3MnSqlp4dcDuf_csfY2h1-i4HV_lnY77guRnt6a0Sy7xxmD-y16jkVXjNHSzCTamDN4VNqExSGoy8nDXDP8fttjk0sZwT0xj_s8LSuK1FzVbcpJgo9nIxLBYZTWmv57HjYA0q0IJLZyCLJWF4xDaotW3boxwtEtQ/s919/1-4-tanh-1.jpg)

The tanh function for z = 3,02

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg4ZwR_uymBs_3vyj_xIXTP9bj9FSD4sQCyyIi487ZMHR3x2O4jud4_9lHWnonMDgnSn-CvPVyvLRquGP9fGyVWQGN1-7DsCbVmtsokvg-ZXiZKkUEI1cZCKTIjkl9Ou2hPA6g8b9Zb8v5uGLmK_lWYwNCbul6_6dJH1IVAlZnJipMqX37oHw60kZRmY8s/w640-h80/1-4-tanh-2.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg4ZwR_uymBs_3vyj_xIXTP9bj9FSD4sQCyyIi487ZMHR3x2O4jud4_9lHWnonMDgnSn-CvPVyvLRquGP9fGyVWQGN1-7DsCbVmtsokvg-ZXiZKkUEI1cZCKTIjkl9Ou2hPA6g8b9Zb8v5uGLmK_lWYwNCbul6_6dJH1IVAlZnJipMqX37oHw60kZRmY8s/s818/1-4-tanh-2.jpg)

The formula for sigmoid function is:

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAs-QqW36L1knHMyQJUqhHKLkvaFjzM2v1Cf4EgqMPdHhD5C3ly2oDS_PStNIjaxBZykMwaEUrXNZuOWYBkl_FZpvNyw2eUZ0r5ky60p5z_taOy6BejK1_vb6WpJUEQ8TbeA95vSOljAkZAR8TszrY0f-hpKhGrIe4NwJqVuE27hdtag770YkVgkg-5Qs/w640-h84/1-4-sigmoid-1.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAs-QqW36L1knHMyQJUqhHKLkvaFjzM2v1Cf4EgqMPdHhD5C3ly2oDS_PStNIjaxBZykMwaEUrXNZuOWYBkl_FZpvNyw2eUZ0r5ky60p5z_taOy6BejK1_vb6WpJUEQ8TbeA95vSOljAkZAR8TszrY0f-hpKhGrIe4NwJqVuE27hdtag770YkVgkg-5Qs/s790/1-4-sigmoid-1.jpg)

The sigmoid function for z = 3,02

The symbol σ represents sigmoid function.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg037DKdEdrenzQdfB6nDXJqdE7tQ1VjivAQniM5rr8VtdjTfTS4c8tzkqzWTe6Swml9zR-UCqyJLHBXsENQ0Feo2mFUU9s_JAfagJXQbBhGDxIODeD0WgzR_3cJnkJm4UE9OjQinb7e4zH5ogMT-oC3hKstJdShhQwUxm7g8q1GTl_QrYkXkemI6ZlId0/w640-h66/1-4-sigmoid-2.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg037DKdEdrenzQdfB6nDXJqdE7tQ1VjivAQniM5rr8VtdjTfTS4c8tzkqzWTe6Swml9zR-UCqyJLHBXsENQ0Feo2mFUU9s_JAfagJXQbBhGDxIODeD0WgzR_3cJnkJm4UE9OjQinb7e4zH5ogMT-oC3hKstJdShhQwUxm7g8q1GTl_QrYkXkemI6ZlId0/s819/1-4-sigmoid-2.jpg)

For z=3.02, both the sigmoid and tanh functions return values close to 1, but the tanh function is slightly higher, approaching its maximum of 1 more quickly than the sigmoid.

Which activation function should we use? The ReLU function has largely replaced the tanh and sigmoid functions due to its simplicity, which reduces the required computation cycles. However, if tanh and sigmoid are used, tanh is typically applied in the hidden layers, while sigmoid is used in the output layer.

### Network Impact

A single artificial neuron is the smallest unit of a neural network. The size of the neuron depends on its connections to input nodes. Every connection has an associated weight parameter, which is typically a 32-bit value. In our example, with 4 connections, the size of the neuron is 4 x 32 bits = 128 bits.

Although we haven’t defined the size of the input in this example, let’s assume that each input (x) is an 8-bit value, giving us 4 x 8 bits = 32 bits for the input data. Thus, our single neuron "model" requires 128 bits for the weights plus 32 bits for the input data, totaling 160 bits of memory. This is small enough to not require parallelization.

However, if the memory requirement of the neural network model combined with the input data (the "job") exceeds the memory capacity of a GPU, a parallelization strategy is needed. The job can be split across multiple GPUs within a single server, with synchronization happening over high-speed NVLink. If the job must be divided between multiple GPU servers, synchronization occurs over the backend network, which must provide lossless, high-speed packet forwarding.

Parallelization strategies will be discussed in the next chapter, which introduces a Feedforward Neural Network using the Backpropagation algorithm, and in later chapters dedicated to Parallelization.

### Summary

Deep Learning leverages Neural Networks, which consist of artificial neurons. An artificial neuron mimics the structure and operation of a biological neuron. Input data is fed to the neuron through connections, each with its own weight parameter. The neuron uses these weights to calculate a weighted sum of the inputs, known as the pre-activation value. This result is then passed through an activation function, which provides the post-activation value, or the actual output of the neuron. The activation functions discussed in this chapter are the non-linear ReLU (Rectified Linear Unit), Hyperbolic Tangent (tanh), and logistic Sigmoid functions.

**References:**
* https://nwktimes.blogspot.com/2024/09/ai4ne-ch1-artificial-neuron.html
