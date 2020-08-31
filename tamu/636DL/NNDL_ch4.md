Neural Networks and Deep Learning
---
Chapter 4 | Teaching Deep Learners to Generalize
---

##4.1

##4.2

##4.3 Generalization Issues in Model Tuning and Evaluation

###4.3.1 Evaluating with Hold-Out and Cross-Validation

- Hold-Out
    - avoid reporting overfitting result (because we are using different data set to report scores)
    - simple
    - efficient
    - may underestimate the result if the Hold-Out data set is not "similar" to the training set

- Cross-Validation
    - split the training set to multiple segments, repeatedly test on each of them while training with others,
    use the average score as the final result 
    - result is very close to the true accuracy
    - expensive
    
###4.3.2 Issues with Training at Scale
- When the training set is too large, efficiency becomes a problem
    - Hold-Out and Cross-Validation might not be efficient
- When the model is very complex, hyperparameter-tuing becomes a problem
    - only evaluate in a few epochs instead of the whole training process
    
###4.3.3 Generalization error 
- Data is noisy
- Bad algorithm
- Underfitting(high bias)
- Overfitting(high invariance) -> collect more data