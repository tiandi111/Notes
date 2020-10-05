Neural Networks and Deep Learning
---
Chapter 3 | Training Deep Neural Networks
---

##3.2 Backpropagation


## 3.4 The Vanishing and Exploding Gradient Problem

[Gradient-descent problem] The relative magnitudes of the partial derivatives with respect
to the parameters in different parts of the network tend to be very different, which creates
problems for gradient-descent methods. The problems are vanishing and exploding gradient.

- sigmoid -> vanishing gradient 
- large weight -> exploding gradient
- relu -> dead neurons
- leaky relu
- maxout -> more parameters

##3.5 Gradient-Descent Strategy

- Learning Rate Decay
    - Exponential Decay
    - Inverse Decay
- Momentum
    - Normal Momentum
        - Accelerate learning
        - avoid local optimum
        - avoid gradient slowdown (flat region)
    - Nesterov Momentum
        - incorporate lookahead information
- Parameter-Specific Learning Rate
    - AdaGrad
- Cliffs and Higher-Order Instability
    - cliff in loss surface makes it harder to choose step size
        - small step size -> converge too slow
        - large step size -> exploding gradient, miss optimum
    - first-order derivative provides insufficient information about loss surface
- Gradient Clipping(梯度修剪)
    - useful in preventing gradient explosion
    - Value-based clipping
        - min clip
        - max clip
    - Norm-based clipping
        - L2-normalization
- 

## 3.6 Batch Normalization
   
- Covariate shift
    - Covariate shift
    
        The distribution of input change 
        
    - Inner covariate shift
        
        The distribution of hidden-layer output changes as the parameter changes.
       
- Batch Normalization
    - reduce covariate shift
    - use higher learning rate 
    - be less careful about param initialization

## 3.7 Practical Tricks for Acceleration and Compression

### 3.7.1 GPU Acceleration
- GPUs are faster at matrix operations (by single-instruction-multiple-threads)
- GPUs have larger memory bandwidth (it can be a bottleneck)

### 3.7.2 Parallel and Distributed Implementations
- Hyperparameter parallelism

    Training with different set of hyperparameters in parallel.
    
- Model Parallelism
    
    When model is too large, it can be parallelized on multiple GPUs.

- Data Parallelism
    
    When model is small enough to fit in memory but the data set is too large.

    - Parameter server architecture
    - sync, async update

- Hybrid Parallelism

### 3.7.3 Algorithmic Tricks for Model Compression
- Sparsifying Weights in Training
    - drop small links(weights) since they have minor influence on the result
    - use L1, L2 regularization
    - Huffman coding
    - quantization

It's necessary to fine-tune the sparsified model for a couple of epochs.

- Leveraging Redundancies in Weights

- 