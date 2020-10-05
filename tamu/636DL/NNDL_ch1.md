Neural Networks and Deep Learning
---
Chapter 1 | Introduction
---

## 1.2 The Basic Architecture of Neural Networks
### 1.2.1 Single layer: Perceptron
- Objective Function
- Why Introduce Activation Function
    - It introduces nonlinearity
    - Some of them squashes input values which can be used as probability
- Choose Loss Function 

### 1.2.2 Multilayer Neural Networks

## 1.5 The Secrets to the Power of Function Composition

[Function Composition]Much of the power of deep learning arises from the fact that repeated composition of multiple nonlinear functions has
significant expressive power. 

Much of the power of deep learning arises from the fact that the repeated composition of certain types of functions increases
the representation power of the network, and therefore reduces the parameter space required for learning.

- Increase the representation power with function composition
    - Neural Networks with only linear activations does not gain from increasing the number of layers of it
    - Activation functions enable nonlinear mappings of the data, so that the embedded points can become linearly separable.
- Decrease parameter requirement with depth
    
## 1.6 Common Neural Architectures
- Most of the basic machine learning models can be simulated by a neural network with no more than two layers
- Radial Basis Function Networks(径向基函数网络)
    - Wide Neural Networks
        - two layers: first for unsupervised learning, second for supervised learning
        - can be thought of supervised nearest-neighbor method
    - gain representaion power by expanding feature space rather than 
- Restricted Boltzmann Machines
- Recurrent Neural Networks(循环神经网络)
    - designed for sequential data like text sentences, time-series, and other discrete sequences like biological sequences
    
