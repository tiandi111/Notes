### Deep Learning High-level IR comparison —— Static Graph vs Dynamic Graph vs Programming Language

- Static Graph
    - What
    
        Represent deep learning model as a static computational graph. Compile the graph ahead-of-time then run it.
        Keyword: Declarative programming paradigm; Define-and-run; Two-step approach(compilation and execution).
    
    - Good
        - Less compilation overhead. 
            
            The graph is only compiled once.
        
        - Tolerant to bad compilation algorithm.
        
            The one-time graph compilation overhead can be amortized with large enough training(inference) samples.
        
        - Easy to schedule computation across different hardwares (Why?)
        - Better performance.
            
            Fixed computational graph at runtime, more opportunity to optimize.
        
    - Bad
        - Less expressive.
        
            Hard to implement models with variable input structure(e.g, RNNs) and models with complex control flow
            (e.g, recursive tree model). Because the computational graph is fixed at runtime.
        
        - Hard to Debug.
        
            The model is "Define-and-Run". Hard to debug at runtime.
        
        - Hard to incorporate dynamic features.
        
            It is possible for the static computational graph approach to implement dynamic features such as: dynamic
            input shapes, control flows etc. However, incorporate these features into the computational graph implementation
            is hard.  
            
    - Example
        - Tensorflow
        - Theano
            
    - Summary
        
        Static computational graph get high performance at the expense of expressiveness and ease of use.
        
- Dynamic Graph
    - What
    
        The computational graph is defined as it is running. Usually implement data-flow part and leave the control-flow
        part to the host language. 
        Keyword: Imperative programming paradigm; Define-by-run; Single-step approach(execution).
    
    - Good
        - More expressive.
            
            Graph is constructed each time it is run, hence, variable input structure and control flow can be supported.
            
        - Easy to develop and debug(for users).
        
            Users can debug the model as programming languages.
            
        - Easier to implement(for tookit designer). 
                
            Since the control flow part is usually leaved for host language.  
            
    - Bad
        - More compilation overhead
        
            The graph must be compiled every time it is run.
            
        - More performance challenges.
        
            Less optimization opportunity due to the lack of graph knowledge. Memory management challenge.
            
    - Example
        - Pytorch
        - Chainer
        - DyGraph
        
    - Summary
    
        Dynamic computational graph makes constructing models as easily as programming but faces more performance challenges.
        
- Programming Language
    - What
    
        Domain-specific language that is designed for deep learning. It compensates the instruction stack with common
        deep learning operators while providing control flow support.
    
    - Good
        - 
    - Bad
    - Example
        - TVM-Relay
    - Summary

