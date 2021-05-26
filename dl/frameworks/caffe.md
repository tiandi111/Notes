Caffe
---

Caffe separates the model representation from the actual implementation, and seamless switching between 
heterogeneous platforms furthers development and Caffe can even be run in the cloud.

Highlights
---
- Modularity
    - allow easy extension to new data formats, network layers and loss functions
- Separation of representation and implementation
    - Caffe model definitions are written as config files using Protocol Buffer language
    - Upon instantiation, Caffe reserves exactly as much memory as needed for the network
    - abstracts from its underly- ing location in host or GPU. Switching between a CPU and GPU implementation is exactly
     one function call.
- Test coverage
- Python and MATLAB bindings
- Pre-trained reference models

Architecture
---
- Data Storage
    - Blob: 4-dimensional arrays
    - 
    - lazy allocation
    
- Layers
    - forward pass
    - backward pass

- Networks and Run Mode
- Training