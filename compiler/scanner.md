Scanner
---

Regular Expression
---
- Three basic operators
    - Union (|)
    - Concatenation (ab)
    - Kleene Closure (*)
    
Finite Automaton
---
- Non-deterministic Finite Automaton (NFA)
    
    An FA that allows transitions on the empty string, ε, and states that have multiple transitions on the same character
    
- Deterministic Finite Automaton (DFA)

    A DFA is an FA where the transition function is single-valued. DFAs do not allow ε-transitions.
    
Development Process
---
```
RE
| Thompson's Construction
v
NFA
| Subset Construction
V  
DFA (DFA Minimization)
|
|  1. table-driven 2. direct-coded 3. hand-coded
V
Code for Scanner
```

- Thompson's Construction
    - only 1 start state and 1 accepting state
    - no transition enters the start state
    - no transition leaves the accepting state
    - An ε-transition always connects two states that were, earlier in the process, the start state and the accepting
     state of nfas for some component res. 
    - each state has at most two entering and two exiting ε-moves, and at most one entering and one exiting move on a
     symbol in the alphabet. 
     
- Subset Construction
    - ε-closure: the set of all states that are reachable from a start state by one or more ε-moves
    - reduce ε-moves iteratively
    
- DFA Minimization
    - group all states that has exactly the same behavior on all characters
    
- Scanner Implementation
    - table-driven
    - direct-coded
    - hand-coded