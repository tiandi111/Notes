Dive into Deep Learning 
---
Chapter 6 | Convolutional Neural Networks
---

###1. From Fully-Connected Layers to Convolutions
- CNNs generate much less parameter than fully-connected MLP(multi-layer perceptron)

- Incorporate two important intuitions
    - translation invariance 平移不变性
    
        识别图像中的物体时，该物体的种类和其所处的位置没有关系（不管猫在图片的哪个位置，都是猫）。
        
    - locality 局部性原理
        
        在较浅的隐藏层中，通常只关注图像中的一个局部范围（提取局部特征）。
        
- Convolution (or cross-correlation) satisfies the above intuitions
    - in a hidden layer, use the same kernel everywhere (treat every patch in the same manner)
    - kernel size smaller than image size (focus on the local region)
    
- Channels on input and output allow our model to capture multiple aspects of an image at each spatial location.
 
###2. Convolutions for Images
- The core operation: a cross-correlation on the 2d input data and the kernel, and then add a bias
- Kernel can learned from data
- Kernel learned from the data is identical with either convolution or cross-correlation
- Feature map and receptive field
    - convolutional layer output is called feature map
    - receptive field refers to all elements(from the previous layers) that may affect the calculation 
    of 感受视野
    - making the network deeper will enlarge the receptive field 

###3. Padding and Stride
- Padding can increase the height and the width of the output.
- Stride can reduce the resolution of the input.

###4. Multi-Channel
- Multiple channels can be used to extend the model parameters of the convolutional layer.
- The 1x1 kernel is equivalent to fully-connected layer applied on a per pixel basis.
- The 1x1 kernel is typically used to adjust the number of channels between network layer and to control
model complexity.

###5. Pooling
- Maximum pooling take the maximum value in the pooling window.
- Maximum pooling take the average value of the elements in the pooling window.
- Pooling layer can alleviate the sensitivity of the convolutional layer to location.
- Maximum pooling, combined with a stride larger than 1 can be used to reduce the spatial dimensions (e.g., width and height).
- Padding and stride can be used in pooling layer.
- The pooling layer's number of output channels is the same as the number of input channels.
