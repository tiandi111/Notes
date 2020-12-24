Hyperparameter Tuning Notes
---
1. Learning rate
    - too large -> gradient exploding -> python Nan

2. Weight decay
    - too large -> gradient vanishing -> loss stop decreasing
        - remember, weights decay at every mini-batch, so make it small
        - [1e-5, 1e-3]
        
3. Weight initialization
    - why random initialization -> 
    break symmetry -> 
    more possible solutions, higher probability to reach global optimum
    (with uniform initialization, there's only one weight set)
    - Tips:
        - pytorch use kaiming_norm as the default, usually it's good