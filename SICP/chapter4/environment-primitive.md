## *environment* & *primitive-procedure*
除了 *eval* 和 *apply* 这两个重要的过程，其他的一些过程也是值得一提的，这里就简略的讲讲。

### *environment*
环境框架在3.2中提到过，若是没什么印象的话可以去稍微看看。 *environment* 其实就是一个环境的实现，书上是用链表实现的，就是现有环境（框架）链接到这个环境的大环境（框架）中。

先让我们来看一下环境要做的事吧：

* 环境中放置了变量名（函数名）的约束，在你请求一个名字时可以返回那个名字所对应的值
* 不同的环境应该不互相冲突，有独立性
* 当你在一个环境中找不到相对应的值时，应该去该环境的大环境中找
* 可以随时增加新环境或者去掉就环境
* 环境中是可以支持随时改变变量名及其约束的

同时我们要注意的一点是，多个小环境可以同时存在而且他们的大环境可能是相同的（递归）。

以上几点表明了环境用链表来实现是相当恰当的，当你想要拓展环境时直接将新环境指针链接向大环境。在环境（框架）中还是得运用链表以支持随意增加或者删减约束的功能(*define*)。

有了上面的分析，要完成 *environment* 中的过程就很简单了，稍微抽象一下怎么实现就看自己喜欢了。（当然貌似scheme没有 *malloc* 什么的，实现也就那么几种方法了。。。）

### *primitive-procedure*
之前我们说过 *eval* 和 *apply* 最终要应用基础过程要用到 *primitive-procedure* 过程（当然你也可以不用内置的，自己写一个同样是可行的），内置过程也并不是可以直接用的。

这是因为当你输入`(+ 2 3)`时你得到的 **+** 并不是 **<proc:+>** 而是 **<char:+>** （当然你可以(define (+ x y) (+ x y))，把+过程手动加入环境中，如果可以这样定义的话（貌似是可以的））不过这样子对于每一个过程都要自己写一个过程，过于麻烦，不如直接调用内置的函数。

调用内置的函数并**不能**够通过调用函数体来实现（因为你根本没有内置函数的函数体），所以要区分开来，因此 *apply* 中的 *primitve-procedure* 子句就是这样来的。

在这里 *primitive-procedure* 就是通过名字来对应基础的过程，然后用 *apply* 过程来将参数应用到函数上。这个是应用基础条件的一个必不可少的过程。（这个貌似就是本身编译器带的 *apply* 。也许不对。。。或许有什么其他方法能实现 *apply* 呢）

具体应用的方法呢就是在最初 *setup-environment* 的时候将基本过程导入，比如: `symbol : + , value : 'primitive +` 那个 *primitive* 是之后加入的，用来区分用户定义的过程，读取出 `list(primitive +)` 之后取出 **+** 然后内置 *apply* 就好。