##*eval* 和 *apply*
求值器中最重要的两个部分就是 *eval* 和 *apply* 了，*eval* 是求值一个给定的表达式，而 *apply* 是应用参数到一个给定的过程，看上去这两个过程没有什么太大的联系，但其实其中的联系是很大的。他们俩共同搭建了求值器的基本架构。

***
现在来分析一下 *eval* 和 *apply*。 

**eval** 的目的就是求解任意一个表达式。为了区分不同的表达式类型，**标识符** 是一定要保留的（比如 *if* 、 *cond* 之类），注意的是，求解任意一表达式其中应该是要有处理将具体实参应用到函数的过程的语句（调用或者内置一个（不过调用会方便得多）），因为在求值这一个表达式之前你 **不知道** 它是一条普通语句还是应用实参，另外还有内嵌在普通语句中的可能，所以处理函数应用的子句是不可少的。当然，你也可以给解释器写一个判断句，但个人感觉这并不如直接内嵌在 *eval* 方便。~~（不过说不定逻辑会更鲜明什么的。。。）~~

再者，还有就是流程跳转、执行顺序、返回值的问题。在拿到一个特殊语句的时候，你就要根据这个语句的类型，来判断这条语句的内部过程接下来该怎么执行、或者是该返回什么，这也许需要写一个 **特殊过程** 。对于进入语句，直接用 *cond* 或者查表就随你喜欢了。

最后再说一下 **环境** ，这是个很容易被忽略的问题（个人认为），因为当你求值一个表达式的时候你 **不可能** 就直接在全局环境中求值。这涉及到变量的作用域、安全性等等的问题。所以必须要追踪到现在的环境，3.2节有和环境相关的内容。

**apply** 的目的是将一组实参应用到一个函数上。这里呢就有拓展环境的问题了，每当一个实际参数应用到流程上时，就创建一个新环境，在拓展的环境中 **绑定实际参数** 。在这个拓展的环境下对参数体的求值就是这个 *apply* 过程的解了。

***
下面是 *eval* 和 *apply* 过程在scheme中的实际分析。

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

### *eval*
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

### *apply*
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

给个例子，比如你在解释器中敲入 `((lambda (x) (* x x)) 5)` *eval* 就会先判断这是不是 *lamba* （后面还会加入其他语句）这里不是就应用到 *application* 上，即为 `(apply (eval(lambda (x) (* x x)) gobal-env) (list-of-value 5 gobal-env))` 这之后就很明显了，先求值函数体也就是 *lambda* 然后返回一个 *procedure* 给 *apply* 做参数，之后将5和 *procedure-parameter* 在新扩展环境绑定之后求值 `(* x x)` 就行了。

### 小结
从整体看呢 *eval* 就如其名就是求值一条语句，或者是一个带着参数的表达式（通过用apply过程）。 *apply* 就是求解一个带着参数的表达式（通过创建一个新的环境）。而 *eval* 中这里没有提到的过程都不会很难理解，比如 *define* 就是将一个变量名（函数名）和一个 *value*（或者转化成 *lambda* 式）绑定、 *if* 就是调用scheme内置的 *if* 等等。这以上就是 *eval* 和 *apply* 的过程。