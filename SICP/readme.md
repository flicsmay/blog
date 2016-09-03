#SICP 
***

这里是我个人对《计算机程序构造和解释》这本书学习中的理解。   

* [第一章](https://github.com/flicsmay/blog/blob/master/SICP/chapter1/chapter%201.md)

* 第四章 
	* 4.1
		* [*eval* & *apply*](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/eval-apply.md)
		* [*environment* & *primitive-procedure*](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/environment-primitive.md)
		* [将语法与执行分离](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/将语法分析与执行分离.md)
	* 4.2
		* [延时求值](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/延时求值.md)
	* 4.3
		* [非确定计算](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/非确定计算.md)
	* 4.4
		* [查询系统的实现](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/查询系统的实现.md)
	* [第四章总结](https://github.com/flicsmay/blog/blob/master/SICP/chapter4/第四章总结.md)

***

###关于编译器
个人是选用drracket，感觉drracket会比mit scheme用起来方便多了。

头文件包是 `#lang planet neil/sicp`。
直接在drracket中输入头文件包然后重新打开就行了。
当代码多时使用drracket会比较卡。这时候就可以注释掉头文件再在其它文件中
`(load path)`就可以导入了，不过要写个安装函数（不然会报错），还是挺麻烦的，有更好的方法也欢迎告诉我。