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
        ```
        DecomposeBCNF(R):
            C = a collection of relations
            if R is not in BCNF:
                find a BCNF violator X->Y, compute the closure of X, that is X+
                split R into two relations, R1 = X+ and R2 = (R-X+) + X, 
                C.add(DecomposeBCNF(R1))
                C.add(DecomposeBCNF(R2))
            else:
                C.add(R)
            return C
        ```
    - The Third Normal Form: For all X->Y, either 
        - X is a super key
        - Y is a prime attribute (there exist FDs inside candidate keys)
        - Recoverable
        - FDs preserving (may introduce redundancy)
        ```
            Decompose3thNF(R, F): R:a relation, F:functional dependencies
                C = a collection of relations
                Let G = ComputeMinBasis(F)
                for each fd X->A in G:
                    C.add(XA)
                if none of relation r in C is a superkey of R:
                    Let one of keys of R, say R', be a schema 
                    C.add(R')
            return C
        ```
    - The Fourth Normal Form: for all non-trivial X↠Y, X must be a super key.
        - i.e, the relation is in BCNF but without MVD(except the special case of FDs)
        - decomposition of 4th NF eliminates both BCNF and 4th NF violators
        ```
            Decompose4thNF(R):
                C = a collection of relations
                if R is not in 4thNF:
                    find a 4thNF violator X->>Y
                    split R into two relations, R1 = X U Y and R2 = X U W, where W is the set of attributes that are not in X and Y
                    C.add(Decompose4thNF(R1))
                    C.add(Decompose4thNF(R2))
                else:
                    C.add(R)
                return C
        ```
        
5. Multi-value dependency
    BCNF cannot eliminate this kind of dependencies.
    
    A multivalued dependency (MVD) X↠Y on a relation R says that if two tuples of R agree on all the attributes of X, then by swapping
    their components in Y, we still get tuples in R, i.e., for each value of X, the values of Y are independent of the values of R\(X∪Y).
    
    - Waste large amount of space
    - Every FDs is an MVD
    
6. Closure
    Computes the closure to infer all FDs.
    
7. Query Processing
    - Preprocessing
        - view processing
        - semantic check
    - Translator
        - remove subquery
            - uncorrelated: first do cross product, then push up the condition above the cross product
            - correlated: the condition involved correlated attributes need to be push up
```
                        token stream           parse tree                  replace the view with its parse tree               expression tree (logic query plan)               
    Program -> Scanner --------------> Parser ------------> Preprocessing --------------------------------------> Translator -----------------------------------> Optimizer ------> ...
```

    
9. Subquery, Join, Aggregation, Modification
    - Subquery
        - Union, Intersection, Difference -> they use !Set semantics!
        - SELECT-FROM-WHERE -> they use !Bag semantics!
        - ALL -> use bag semantics; DISTINCT -> use set semantics
    - Join
        - theta join: join on some conditions
        - natural join: join tuples agreeing on common attributes
        - left outer join: keep all left attributes, match the right with the left on common attributes, discard unmatched rows from the right 
        - right outer join: keep all right attributes, match the left with the right on common attributes, discard unmatched rows from the left 
        - full outer join: keep all left and right attributes, match the right with the left on common attributes 
    - Aggregation
        - GROUP BY: when aggregations used,  any element of SELECT must be either aggregated or in GROUP BY list
        - HAVING: applied only on groups
    - Modification
        - DELETE
            - two stages: mark all tuples that the WHERE clause holds; delete all of them
    
8. Logic Query Plan
    - deal with subquery
        - subquery in FROM clause
            - make logic query plan for subquery
            - do cross product between lqp of subquery and others
        - subqueries in WHERE clause
            - make logic query plan for subquery
            - collect the condition
            - do cross product between lqp of subquery and others
            - combine the new condition with those that originate from the query
    - algebraic laws (we need to first identify the tools on hand that we can use to improve logic query plan)
        - commutate: union, intersect, cross product, natural join, theta join
        - associate: union, intersect, cross product, natural join, theta join (only when attributes satisfy the conditions!!)
    - improve lqp
        - push selection(where clause) down
            - constraint: can only apply on relations that involve all attributes in the selection
            - split selection
            - 
            - pull up then distribute down
        - projection
            - principle: can introduce projection anywhere in an expression tree, as long as it eliminates only attributes that
            are neither used by an operator above nor are in the result of the entire expression
            - 
        - duplicate elimination
        - combine selection and product to equijion
        - grouping associative/commutative operators
            - operators
                - always associative/commutative: natural join, union, intersection
                - under some circumstances: combine natrual join and theta join
    - cost estimation
        - Notation
            - B(R): blocks need to hold the relation
            - T(R): the number of tuples in the relation
            - V(R, a): the value count of the attribute a in the relation R
                - V(R, [a1, a2, ... an]): all combinations of all attributes, that is the number of tuples after eliminating duplicates
            
9. Constraints and Triggers
    - constraints: a constraint is a relationship among data elements that the DBMS is required to enfore
        - keys: unique, not null
        - foreign keys: referential integrity
            - insert or update violation: insert a record that has foreign key not exist in the referenced table
                - must reject
            - delete violation: delete a record causes dangle
                - reject
                - cascade: make the same changes in all related tables
                - set null
        - attribute-based checks: check when insert or update
        - tuple-based checks:
        - cross-relation constraints -> assertion
    - triggers: a trigger is an action only executed when a specified condition occurs 
        - triggers let user decide when to check for a powerful condition
            - assertion is powerful, but the system does not know exactly when to check
            - attr-based and tuple-base checks are checked at known time, but are not powerful
        - row-level | statement-level
            - row-level: execute for each row
            - statement-level: execute only for the statement

10. Bag 
    - Operations: assume t appear n times in R and m times in S
        - Union: tuple t appear n+m times in (R∪S)
        - Intersection: tuple t appear min(n, m) times in (R∩S)
        - Difference: tuple t appear max(0, n-m) times in (R-S)
        - Projection: If the elimination of one or more attributes during the projection causes the same tuple to be created
        from several tuples, these duplicate tuples are not eliminated from the result of a bag-projection.
        - Selection: as like Projection, we do not eliminate duplicates
        - Product: do not eliminate duplicates
        - Joins: do not eliminate duplicates