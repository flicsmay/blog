好，我们来分析一下这个 *eval* 和 *apply* 过程，首先我们要用到 *read* 过程，这也是scheme中的内置过程(可以在p265的脚注222中看到)。就是读取输入的序列然后转换成链表(list)的过程。
例如：

	> (read)
	(+ 3 4)
	(mcons '+ (mcons 3 (mcons 4 '())))

就是把'+ 3 4做成一个链表。
再比如：

	>  (read)
	(/ (+ 1 2) (- 3 4))
	(mcons
	 '/
	 (mcons (mcons '+ (mcons 1 (mcons 2 '()))) (mcons (mcons '- (mcons 3 (mcons 4 '()))) '())))
就是将'/ (+ 1 2) (- 3 4)做成链表，在各自括号下再做成链表。
所以当我们想要进入一个括号中时就只用car、cadr这样往下降就好（上面要是想进入(+ 1 2) 就只用cadr就行了。

好，首先是 *eval* ，这个过程从黑盒子看的话就是拿到一个过程然后产出这个过程求值的结果。从一般情况（无复杂过程） *eval* 中起到关键作用的是倒数第二句:

	((application? exp)
	         (apply (eval (operator exp) env)
	                (list-of-values (operands exp) env)))

这段程序中的application?是用于提出error监测用的，也就是说如果不考虑error情况的话是可以直接默认为这是个可以求值的过程。（上面的语句都是在说明if、cond、之类的情况）比如`(eval (+ 3 4) env)`就是apply + 于 (list 3 4) 这个链表上。(f a b)就是 *apply* f过程 于(list a b)链表中。这里的+ 和 f过程也是通过 *eval* 过程得到的，这个我们稍后再说。这样我们可以得到一个极其简略版的 *eval* :	

	(define (eval exp env)
	  (apply (eval (operator exp) env)
	         (list-of-values (operands exp) env)))

这个过程只做了一件事，就是取出操作符，取出参数列表，然后将操作符应用上去。这时候，你若是将其他过程（gobal-env，apply之类）搭建好的话，你就可以求值最基本的过程了。(其实还要加上一个self-evaluing,这里只是做一个抽象)
当然，仅仅是那么简单的过程是不够的，所以我们就有了 *eval* 那么多的 *cond* 语句。

之后我们就来看 *apply* 过程了，但在看 *apply* 之前，我们要先把 *eval* 过程扩充一下，上面的模型只需要 *apply-primitive-procedure* 就可以了。这时我们就要引入函数的概念，先假设我们的函数只有  *lambda* 过程（但实际上也就是这样的）所有用 *define* 的函数都用 *lambda* +参数表示。这样子我们就可以把 *eval* 中的 *lambda* 子句加入。如下:

	(define (eval exp env)
	  (cond ((lambda? exp)
	         (make-procedure (lambda-parameters exp)
	                         (lambda-body exp)
	                         env))
	        ((application? exp)
	         (apply (eval (operator exp) env)
	                (list-of-values (operands exp) env)))))

在这里， *make-procedure* 就是把当时的lambda函数体和当时的环境捆绑成一个 *procedure* 过程。

现在我们来看看 *apply* 过程，这里只关注函数应用的 *cond* 语句。

	(define (apply procedure arguments)
	  (eval-sequence
	   (procedure-body procedure)
	   (extend-environment
	    (procedure-parameters procedure)
	    arguments
	    (procedure-environment procedure))))

这里就涉及到解释器的一个很重要的点，就是环境框架。这个我们在第3.2节中也看到过的。每当一个过程对象应用于一实际参数时，将构造出一个新框架。原话如下：
>A procedure object is applied to a set of arguments by constructing a frame, binding the formal parameters of the procedure to the arguments of the call, and then evaluating the body of the procedure in the context of the new environment constructed. The new frame has as its enclosing environment the environment part of the procedure object being applied.

这里我们看到*extend-environment*就是完成这件事的`extend-env(<variables> <values> <base-env>)`,在这里我们传递 *procedure* 过程的变量和给定的参数（args）绑定，接下来在之前创造 *procedure* 绑定时的环境上拓展一个新环境，之后把这个环境当成参数递给 *eval-sequence*。
在这里 *eval-sequence* 也就是将lambda中的函数体分别求值。

那这个过程的作用是什么呢？其实求一个 *procedure* （带参数）也就是在一个它的变量和实际参数绑定的环境中求值它的过程体。这也就是 *apply* 和 *eval* 过程中主要做的事。