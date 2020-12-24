PCA
---

1. PCA
    - some concepts
        - design matrix: 
            
            often denoted X, each row represent an individual object
            
        - covariance matrix: 
        
            often denoted S, S = E[ (x-x_mean) (x-x_mean)^T ]
        
    - formulation
        - maximum variance
        - minimum projection error
    - implementation
        - eigenvalue decomposition to covariance matrix
            - covariance matrix is large, so this is inefficient
        - singular value decomposition to design matrix
            - design matrix is small, so more efficient
            
2. Eigenvalue decomposition
    
    For a square matrix with linearly independent eigenvalues, S can be factorized as
    S = QAQ^-1, where Q is the square n*n matrix whose columns are eigenvectors. A is
    a diagonal matrix whose diagonal elements are the corresponding eigenvalues.

3. Singular value decomposition

    A m×n Matrix A can be decomposed as A = U Delta V^T, where U is m×m, Delta is m×n and
    V is n×n. Delta is a diagonal rectangular matrix whose diagonal elements are non-negative.
    The columns of U are the eigenvectors of AA^T.
    The columns of V are the eigenvectors of A^TA.
    
    If A is real, U and V are orthogonal. If we make U, V orthonormal, then
    U^TU = UU^T = I, U^T = U^-1
    V^TV = VV^T = I, V^T = V^-1
   
    SVD is not unique because we can reorder and scale the diagonal elements of Delta.