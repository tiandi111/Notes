I/O Cheatsheet
---

- library: iostream
- standard input: std::cin, buffered
- standard output: std::cout, buffered
- standard error: std::cerr, non-buffered
- standard log: std::clog, buffered
- output operator: <<
    - example： std::cout << "Hi" << std::endl;
    - return the left-hand operand, hence the "<<" can be chained
- input operator: >>
    - example： std::cin >> a >> b;
    - return the left-hand operand, hence the ">>" can be chained
- manipulator: endl
    - end the current line
    - flush the buffer
    


