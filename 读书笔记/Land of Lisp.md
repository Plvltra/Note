[The Roots of Lisp](http://daiyuwen.freeshell.org/gb/rol/roots_of_lisp.html#tex2html2)

#### 特性
- Functional
- Macro
- Restart
- Generic Setter
- DSL
- CLOS(common lisp object system)
- Continuation
- Brevity
- Multicore
- Lazy

## Section 1
#### 3. EXPLORING THE SYNTAX OF LISP CODE
- Syntax: The syntax of a piece of text represents the basic rules that it needs to follow to be a valid sentence
- Semantic: semantic is what the sentence actually means.(例如我们可以用不同语言的语法表达同一内容)
- **Having a simple syntax is a defining feature of the Lisp language.**
<br>

- Symbols in Common Lisp are case-insensitive(avoid using uppercase)
- Data Mode: Placing a quote in front of lists so that they won’t be evaluated as a command is called *quoting*. 