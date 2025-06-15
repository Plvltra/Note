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
- 替换alist value的技巧:  *push* new items onto the list, only the most recent value will be reported by the *assoc* function
``` lisp
;BUILDING A TEXT GAME ENGINE

;;;;;; Describing the Scenery with an Association List
(defparameter *nodes* '((living-room (you are in the living-room. a wizard is snoring loudly on the couch.))
                        (garden (you are in a beautiful garden. there is a well in front of you.))
                        (attic (you are in the attic. there is a giant welding torch in the))))

;;;;;; Describing the Location
(defun describe-location (location nodes)
  (cadr (assoc location nodes)))
; (describe-location 'living-room *nodes*)
; (YOU ARE IN THE LIVING-ROOM. A WIZARD IS SNORING LOUDLY ON THE COUCH.)

;;;;;; Describing the Paths
(defparameter *edges* '((living-room (garden west door)
                                     (attic upstairs ladder))
                        (garden (living-room east door))
                        (attic (living-room downstairs ladder))))
(defun describe-path (edge)
  `(there is a ,(caddr edge) going ,(cadr edge) from here.))
; (describe-path '(garden west door))
; (THERE IS A DOOR GOING WEST FROM HERE.)

; Describing Multiple Paths at Once
(defun describe-paths (location edges)
  (apply #'append (mapcar #'describe-path (cdr (assoc location edges)))))
; (describe-paths 'living-room *edges*)
; (THERE IS A DOOR GOING WEST FROM HERE. THERE IS A LADDER GOING UPSTAIRS FROM HERE.)

;;;;;; Describing Objects at a Specific Location
; Listing Visible Objects
(defparameter *objects* '(whiskey bucket frog chain))
(defparameter *object-locations* '((whiskey living-room)
                                   (bucket living-room)
                                   (chain garden)
                                   (frog garden)))

(defun objects-at (loc objs obj-locs)
  (labels ((at-loc-p (obj)
                     (eq (cadr (assoc obj obj-locs)) loc)))
    (remove-if-not #'at-loc-p objs)))
(objects-at 'living-room *objects* *object-locations*)
; (WHISKEY BUCKET)

; Describing Visible Objects
(defun describe-objects (loc objs obj-locs)
  (labels ((describe-obj (obj)
                         `(you see a ,obj on the floor.)))
    (apply #'append (mapcar #'describe-obj (objects-at loc objs obj-locs)))))
; (describe-objects 'living-room *objects* *object-locations*)
; (YOU SEE A WHISKEY ON THE FLOOR. YOU SEE A BUCKET ON THE FLOOR.)

;;;;;; Describing It All
(defparameter *location* 'living-room)

(defun look ()
  (append (describe-location *location* *nodes*)
    (describe-paths *location* *edges*)
    (describe-objects *location* *objects* *object-locations*)))

;;;;;; Walking Around in Our World
(defun walk (direction)
  (let ((next (find direction
                  (cdr (assoc *location* *edges*))
                :key #'cadr)))
    (if next
        (progn (setf *location* (car next))
               (look))
        '(you cannot got that way.))))
; (walk 'west)

;;;;;; Picking Up Objects
(defun pickup (object)
  (cond ((member object (objects-at *location* *objects* *object-locations*))
          (push (list object 'body) *object-locations*)
          `(you are now carrying the ,object))
        (t '(you cannot get that.))))
; (pickup 'whiskey)

;;;;;; Checking Our Inventory
(defun inventory ()
  (cons 'items- (objects-at 'body *objects* *object-locations*)))
; (inventory)

```

#### 6: Reading and Printing in Lisp
- print(prin1是print的简洁版本, 没有换行和结尾的空格): 输出内容需要quote符号括起
- read: 输入内容需要quote符号括起
- ![[Computer Science/Lisp/attachments/2ff8bc01ce3d47c783b03ebc57fced2b_MD5.jpeg]]
- princ: 以适合人类读的方式输出
	```lisp
	; print vs princ
	(print '3) => 3         An integer
	(print '3.4) => 3.4     A float
	(print 'foo) => FOO     A symbol 
	(print '"foo") => "foo" A string
	(print '#\a) => #\a     A character

	(princ '3) => 3
	(princ '3.4) => 3.4
	(princ 'foo) => FOO
	(princ '"foo") => foo
	(princ '#\a) => a
	```
- character表示方式: #\开头, 例如#\a #\newline  #\tab #\space
- *homoiconic*: a programming language that uses the same date structures to store data and program code
- *quote*: 'foo 是 (quote foo) 的缩写,  意味着(list 'quote x)可以产生(quote x), 也就是'x
- coerce
```lisp
;;;;;; game-repl
(defun game-read ()
  (let ((cmd (read-from-string
               (concatenate 'string "(" (read-line) ")"))))
    (flet ((quote-it (x)
                     (list 'quote x)))
      (cons (car cmd) (mapcar #'quote-it (cdr cmd))))))
; (game-read)
; walk east
; (WALK 'EAST)

(defparameter *allowed-commands* '(look walk pickup inventory))
(defun game-eval (sexp)
  (if (member (car sexp) *allowed-commands*)
      (eval sexp)
      '(i do not konw that command.)))

(defun tweak-text (lst caps lit)
  (when lst
        (let ((item (car lst))
              (rest (cdr lst)))
          (cond ((eq item #\space) (cons item (tweak-text rest caps lit)))
                ((member item '(#\! #\? #\.)) (cons item (tweak-text rest t lit)))
                ((eq item #\") (tweak-text rest caps (not lit)))
                (lit (cons item (tweak-text rest nil lit)))
                ((or caps) (cons (char-upcase item) (tweak-text rest nil lit)))
                (t (cons (char-downcase item) (tweak-text rest nil nil)))))))
(defun game-print (lst)
  (princ (coerce (tweak-text (coerce (string-trim "() "
                                                  (prin1-to-string lst))
                                     'list)
                             t
                             nil)
                 'string))
  (fresh-line))
; (game-print '(not only does this sentence have a "comma," it also mentions the "iPad."))
; Not only does this sentence have a comma, it also mentions the iPad.

(defun game-repl ()
  (let ((cmd (game-read)))
    (unless (eq (car cmd) 'quit)
      (game-print (game-eval cmd))
      (game-repl))))

```

#### 6.5: Lambda
- lambda: 是一个macro, macro允许参数不被立即求值
- *higher-order functional programming*: Many functions in Lisp accept functions as parameters. If you use these functions, you are using a technique called higher-order functional programming.