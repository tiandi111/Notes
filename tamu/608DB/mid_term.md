Review
---

1. We don't expect Redundancy in database schemas
    - Waste space
    - cause inconsistency
    
2. How redundancy causes inconsistency?
    - Update anomaly
        - update one place but forget to update others
    - Deletion anomaly
        - delete all related rows will lose information

3. What really cause the redundancy?
    - there is non-super key determines others attributes<br/>
        
        Think it this way, with only super key determines every other attributes,
        no redundancy will exist because the union of those attributes in super key 
        is guaranteed to be unique. 
        
4. Normal Form
    - BC Normal Form: For all non-trivial X->Y, X must be a super key. 
        - BC normal form is too restricted, it will lose some FDs, which leads to the 3th NF,
        - Recoverable
        - Not necessarily FDs preserving
        - eliminate FD redundancy (but not necessarily all redundancies)
    - The Third Normal Form: For all X->Y, either 
        - X is a super key
        - Y is a prime attribute (there exist FDs inside candidate keys)
        - Recoverable
        - FDs preserving (may introduce redundancy)
    - The Fourth Normal Form: for all non-trivial X↠Y, X must be a super key.
        - i.e, the relation is in BCNF but without MVD(except the special case of FDs)
        - decomposition of 4th NF eliminates both BCNF and 4th NF violators
        
5. Multi-value dependency
    BCNF cannot eliminate this kind of dependencies.
    
    A multivalued dependency (MVD) X↠Y on a relation R says that if two tuples of R agree on all the attributes of X, then by swapping
    their components in Y, we still get tuples in R, i.e., for each value of X, the values of Y are independent of the values of R\(X∪Y).
    
    - Waste large amount of space
    - Every FDs is an MVD
    
6. Closure
    Computes the closure to infer all FDs.
    
7. Query Processing
                        token stream           parse tree               expression tree (logic query plan)               
    Program -> Scanner --------------> Parser ------------> Translator -----------------------------------> Optimizer ------> ...
    
    

8. Logic Query Plan
        