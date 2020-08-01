Rejection Sampling
---
Definition:
Acceptance-Rejection sampling is a way to simulate random samples from an unknown (or difficult to sample from) distribution 
(called the target distribution) by using random samples from a similar, more convenient probability distribution. A random 
subset of the generated samples are rejected; the rest are accepted. The goal is for the accepted samples to be distributed 
as if they were from the target distribution.

人话：
拒绝采样简单说就是用一个好的概率分布（好生成的，通常是均匀概率分布）去模拟一个不太好的概率分布（不好生成或者不好采样）。怎么做呢，我们先通
过一定的方法把好概率分布等比例放大，使得它能够在pdf上"包住"我们要模拟的概率分布。然后我们进行采样，如果采样到的点落在了要模拟的概率分布pdf
内，则接受，否则拒绝。