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

## Section 1 Lisp Is Power
#### 3. EXPLORING THE SYNTAX OF LISP CODE
- Syntax: The syntax of a piece of text represents the basic rules that it needs to follow to be a valid sentence
- Semantic: semantic is what the sentence actually means.(例如我们可以用不同语言的语法表达同一内容)
- **Having a simple syntax is a defining feature of the Lisp language.**
<br>

- Symbols in Common Lisp are case-insensitive(avoid using uppercase)
- Data Mode: Placing a quote in front of lists so that they won’t be evaluated as a command is called *quoting*. 

##### Cons Cells
- A cons cell is made of two little connected boxes
- Lists in Lisp are just an abstract illusion—**all of them are actually composed of cons cells**

##### Lists in Lisp
- Lisp will always go out of its way to “hide” the cons cells from you. When it can, it will show your results using lists. It will show a cons cell (with the dot between the objects) only if there isn’t a way to show your result using lists.
- 以下三种创建一个list的方式等价
	- (CONS 'PORK (CONS 'BEEF(CONS 'CHICKEN ())))
	- (LIST 'PORK 'BEEF 'CHICKEN)
	- '(PORK BEEF CHICKEN)

## Section 2 Lisp Is Symmetry
#### 4. Making Decisions with Conditions
- 条件分支
	- If: 仅有一个分支被Evaluate(可以用progn在单个分支使用多条语句)
		- 返回某个分支(不返回值是为了)
	- When, Unless(相当于If not): 更加通用, 且自带progn
	- Cond: 更加通用, 且分支自带progn
	- case
	- and, or
- comparing: eq, equal, and more
	- **Conrad’s Rule of Thumb**
		- Use `eq` to compare two symbols
		- Use `equal` for everything else
	- eq: 比较两个对象地址是否相等
	- equal: it will tell you when two things are isomorphic, meaning they “look the same.” 
	- eql: 在eq的基础上附带比较numbers和characters
	- equalp: 在equal基础上加了一些特殊情况的处理(例如: 可以判等不同大小写的string, 整数与浮点数)
	- 其他的比较: 在equal基础上增加了对特殊数据类型处理(例如: =(equal sign) 支持numbers的处理)

#### 5: Building a Text Game Engine
- *Using lists and symbols as an intermediary for manipulating text* is an old-school Lisp technique. However, it can often lead to very elegant code, since list operations are so fundamental to Lisp
- 可以使用 `association list (alist)`存储键值对关联节点, 使用`assoc`根据key查询
- `quasiquoting`特性: allows us to create chunks of data that have small pieces of Lisp code embedded in them.
	```lisp
	`(there is a ,(caddr edge) going ,(cadr edge) from here.)
	```
- `higher-order functions`:  common-lisp使用#'传递函数
	```lisp
	> (mapcar #'car '((foo bar) (baz qux)))
	(foo baz)
	
	#'car相当于(function car)
	> (mapcar (function car) '((foo bar) (baz qux)))
	(foo baz)
	```
- `predicates`函数命名规则:  When a function returns nil or a truth value, it’s a Common Lisp convention to append a p to the end of that function’s name.(例如oddp)