Review
---

1. Perceptron(Linear separator)
    - Hypothesis set: sign(W^Tx)
    - Learning algorithm: PLA
        - idea: start with some weights and try to improve it
        - algorithm:
        ```
        for i in max_iter:
            for (x, y) in in_samples:
                if sign(W^Tx) != y:
                    all_mispred_samples.append((x, y))
            for (x, y') in all_mispred_samples：
                W_(i-1) += y * x
        ```
      - pocket algorithm: use the model with the smallest in-sample error
    - Theoretical basis: If the data can be fit by a linear separator, then after some finite number of steps, PLA will find one.
    
2. Linear Regression
    - y = W^Tx, E = (y-W^Tx)^2
    - Has analytic solution:
        - Normal equation: take the derivative of E, we get X^TXw = X^Ty
        - The linear regression algorithm gets the smallest possible Ein in one step
        
3. Logistic Regression(for classification task)
    - predict the probability with: sigmoid(W^Tx)
    - sigmoid: 1 / (1 + e^(-s))
    - Cross entropy loss(for binary case): E = ln(1 + e(-y * W^T * x))
        - y takes {+1, -1}, y is the real label
    - Cross entropy is an alternative representation of maximum likelihood estimation, the loss function used in logReg
    is actually derived from MLE
    
4. Softmax Regression(for multi-class classification)

5. Gradient Descent
    - Batch-GD: update for the entire batch
    - SGD: update for each sample
    - Mini-Batch GD: update for each mini-batch (accumulate loss -> compute gradient -> update)
    
6. CNN
    - Domain knowledge
        - translation invariance
        - locality
    - Basics (hk is the size of the kernel) 
        - Conv
            - H(i+1) = H(i) + hk -1 
        - Padding 
            - "same" padding (or half-padding): Ph = (hk - 1)/2 on each side
            - "valid" padding: not use any padding...
                - without padding, pixels on boarders are under-represented
            - "full" padding: Ph = hk - 1 on each side, increase by hw-1, useful when doing reverse convolution
        - Strides (Sk is the stride)
            - H(i+1) = (H(i) - hk) / Sk + 1
        - Bias
            - one for each channel, added on each pixel
        - Pooling
            - diff with conv: apply on each feature map, does not change the number of feature maps
            - with stride of 1, H(i+1) = H(i) + hk -1 
            - provide nonlinearity and translation invariance
            - global average pooling before FC to reduce computation load
        - FC
            - 
        - Skip connection
            - provide unimpeded gradient flow
            - provide multiple level of abstractions and let the network itself to decide which level to use; the network
            becomes a "bag" of different models, similar to ensemble
            - the above point can be seen from an optimization perspective, in which says that deeper networks have more
            complicated loss surface and require much more time and more sophisticated optimization techniques to converge.
            The skip connections ease this difficulty by allowing the network to converge at "less-representative" minima
