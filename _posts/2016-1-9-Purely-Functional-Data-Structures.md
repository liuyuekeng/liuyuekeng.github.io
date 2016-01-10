---
layout: post
title:  "翻译《Purely Functional Data Structures》"
date:   2016-1-9 17:14:00
categories: [books, coding]
---

#1 简介

对高效数据结构的研究已经超过30年，这些牛逼的程序员从各种复杂问题中提取出高效的解决方案，产生一揽子著作。
本来这些著作应该不依赖于编程语言的，但不幸的是，这些著作看上去更像是亨利福特在说
“程序员能够使用任意的编程语言，只要它指令式的（这里把imperative翻译成指令式，后面都是如此）”。
（亨利福特的名言：“[Customers] can have any colorthey want, as long as it’s black.”）
是只有少数的数据结构是适合在函数式语言中使用的，诸如Standard ML,Haqskell。
这本书就是要来帮助弱小的!思考，分析函数式数据结构的设计。

##1.1 函数式 VS. 指令式数据结构

函数式编程方法论的优势已经为大众所知[Bac78, Hug89, HJ94]，但是绝大多数的程序仍然是用指令式编程语言（C语言之类）写的。
这种明显的矛盾可以轻易地用函数式语言起步比较晚来解释，但这个差距也在逐渐缩小。
函数式语言从基础编译技术，到复杂的分析和优化，各个方面都有了长足的进步。
然而，函数式编程还有一块短板，没多人能够用上合适的数据结构。
不幸的是，没多少书可以在这方面提供有用的参考。

为什么函数式的数据结构比起指令式的数据结构，更难设计和实现呢？
主要是两个最基本的问题。

第一个问题是，函数式编程强烈反对破坏性的更新（事务），这点非常难搞，相当于没收了大厨的刀子。
其实破坏性的更新就像刀子，可以很危险，也可以很高效。
指令式的数据结构本身极度依赖事务，所以函数式编程只能另谋出路。

第二个问题是，函数式数据结构需要比指令式的数据结构更加灵活。
尤其是，当我们更新一个指令式的数据结构的时候，往往认为旧版本的数据可以不要了，
但更新一个函数式的数据结构的时候，我们希望新的和旧的数据都支持访问，来实现更多的功能。
一个数据结构支持多版本访问，我们称它为具有不变性，
反之同时支持持单版本访问的叫做短暂性（好吧，这是我随便起的，后面都这么翻译的，原文是ephemeral）[DSST89]。
函数式编程语言有一个特点就是，所有数据结构生来就是具有不变性的。
指令式的数据结构基本上都是短暂性的，但是，如果指令式编程语言需要一个具有不变性的数据结构时，
理所当然的这个数据结构会比较复杂，而且相比起来效率更低。

另外，理论家已经至少证明，在一些情况下，函数式编程语言会比指令式编程语言运行效率低[BAG92, Pip96]
对于这些观点，这本书要打他们的脸，性能逼近短暂性数据结构的函数式数据结构往往是可以搞得出来的。

##1.2 严格计算 VS. 惰性计算

大多数（顺序的）函数式语言可以根据他们求值的顺序，被归为严格计算和惰性计算两类。
这是一个一直被函数式程序员们虔诚热烈讨论的话题。
两者最明显的区别莫过于他们如何处理函数的参数。
在严格计算的语言中，参数在函数体执行前被求值。
而在惰性计算语言中，参数是在需要的时候才被求值的。
他们在一开始传递的时候是未被赋值的，当且仅当计算需要时才被求值。
另外，一旦参数被赋值，参数的值就被保存起来了，一旦再次需要该值的时候，就可以不经过重复计算直接找到。
这个保存的行为被称为记忆[Mic68]。

不同的求值顺序有各自的优缺点，但严格计算模式至少有一个显而易见的优势：
减轻了渐进的（asymptotic什么鬼）复杂性。
在严格计算的语言中，能够准确说出任意表达式何时会被求值，在大部分情况下更加显然。
因此，推测给定程序的运行情况也就更直观。
然而，在惰性求值的语言中，即便是老手也很难预测一个表达式什么时候，甚至是否会被求值。
使用这种语言的程序员常常不得不假装语言本身是严格的，来估算总运算时间。

两种求值方式都间接地影响着数据结构的设计和分析。
正如我们将要在第三、四章中看到的，严格计算的语言能够描述数据结构的下界（这里把worst-case data structures翻译成数据结构的下界，后面都将worst-case翻译为下界），
却不能描述摊销的数据结构，而惰性计算语言能够描述摊销数据结构，却不能表示数据结构的下界。
为了能够描述这两种数据结构，我们需要一种支持两种求值顺序的语言。
幸运的是，将两者融合到一个语言中并不难。
第二章介绍了$符，往严格计算语言中添加惰性计算的方法（以Standar ML为例子）

##1.3 贡献

参考了这些文献，这就不翻译了吧

* Functional programming. Besides developing a suite of efficient data structures that
are useful in their own right (see Table 1.1), we also describe general approaches to
designing and analyzing functional data structures, including powerful new techniques
for reasoning about the running time of lazy programs.

* Persistent data structures. Until this research, it was widely believed that amortization
was incompatible with persistence [DST94, Ram92]. However, we show that memoization,
in the form of lazy evaluation, is the key to reconciling the two. Furthermore, as
noted by Kaplan and Tarjan [KT96b], functional programming is a convenient medium
for developing new persistent data structures, even when the data structure will eventually
be implemented in an imperative language. The data structures and techniques in
this thesis can easily be adapted to imperative languages for those situations when an
imperative programmer needs a persistent data structure.

* Programming language design. Functional programmers have long debated the relative
merits of strict and lazy evaluation. This thesis shows that both are algorithmically important
and suggests that the ideal functional language should seamlessly integrate both.
As a modest step in this direction, we propose $-notation, which allows the use of lazy
evaluation in a strict language with a minimum of syntactic overhead.

##1.4 源码

所有的源码都用Standard ML写的[MTH90]，提供原始的惰性计算。
而且能够方便地转化成任何支持严格和惰性计算的函数式编程语言。
对那些使用纯严格计算或纯惰性计算函数式语言的程序员来说，就只有部分的数据能用得上。

在第七章第八章中，我们将会遇到一些递归的数据结构，用Standard ML很难描述清楚。
由于语言的限制，对于某些复杂的递归形式，比如多态递归和递归模块。
当这种情况出现的时候，我们会牺牲可执行性，使用类似Standard ML的伪代码来将递归形式描述清楚。
然后我们才来看怎么实现成合法的Standard ML。
这些例子可以被看做对编程语言设计界提供具有简单抽象能力语言的挑战。

##1.5 术语

任何关于数据结构的讨论都会充满歧义，毕竟数据结构这个词就有至少四中相关的有意义的释义。

* 一个抽象数据类型（一个类和它的方法的集合）。我们将它当成一个抽象

* 一个具体抽象数据类型的实现。我们把它当成一个实现。
但要注意，一个实现不一定非要是代码实现，一个实现的设计也算。

* 一个数据类型的实例。比如某种列表或某种树。
我们将它当做一个实例，通常来说是一个对象或者一个版本。
然而，特定的数据类型通常有自己的名称。
举个例子，我们将简单地把一个栈或队列对象当做叫做栈，或队列。

* 一个不变的标识是在更新中保持不变。
比如说，一个基于栈的解析器，我们经常随口讲“the stack”，就好像只有一个唯一的栈，
而不是在不同的时间存在不同的版本。
我们将这种标识当做永恒不变的。、
这种情况主要出现在我们讨论具有不变性的数据结构的时候，
我们所说的同一个数据结构的不同版本，其实是指不同版本共享一个共同的不变标识

Roughly speaking, abstractions correspond to signatures in Standard ML, implementations
to structures or functors, and objects or versions to values. There is no good analogue for
persistent identities in Standard ML.（妈的，这段实在不知道是什么鬼）

类似地，operation这个词是被重载的，表示一个抽象数据类型支持的功能，和这些功能的实现。
我们的operation指的是第二个含义，对于第一个含义我们使用operator和function做代替。

##1.6 概览

这本书包括两个部分。第一个部分（第二章到第四章）内容涉及算法方面的惰性计算。
第二章简要回顾了懒惰计算的基本概念，引入了$运算符。

第三章是这本书余下部分的基础。它描述了惰性运算在结合摊销和不变性中的地位，
而且给定了两种分析惰性计算摊销的消耗的方法。

第四章说明了在一种语言中结合严格计算和惰性计算的力量。
它描述了通过系统地调度惰性组件提前执行，常常能够在摊销数据结构中提取出下界。

本书的第二部分（从第五章到第八章）涉及函数式数据结构的设计。
记载适合每一种目的的高效数据结构是一个无法完成的任务，
所以我们把心思集中在几种常见的设计高效函数式数据结构的技术，
并列举每一种技术的一个或几个基本抽象的实现。
比如优先队列，随机存取结构，和各种序列。

第五章描述了惰性重建，这是全局重建的一种惰性变形[Ove83]。
惰性重建比全局重建明显简单，但是收益摊销而不是下界。
结合惰性重建和第四章中的调度技术，下界可以被找出来。

第六章探讨了数字的表示，实现了类似数字表示的设计（通常是二进制数字）。
在这个模型中，设计高效的插入和删除操作，相当于选择增加或减少常数时间的二进制数字变体。

第七章考察数据结构的引导[Buc93]。数据结构引导有两种：
结构分解，无限的解决方案通过有限的解决方案引导而来；
结构抽象，高效的解决方案通过抵消的解决方案引导而来。

第八章介绍了隐含的递归减缓，Kaplan and Tarjan [KT95]递归减缓技术的惰性变种。
就惰性重建而言，隐含的递归减缓要更为简单，但是收获摊销而不是下界。
同样的，我们可以通过调度来解决下界的问题。

最后，第九章通过持久化数据结构，编程语言设计，和一些公开问题的讨论，
总结了上述工作对函数式编程的影响。

#2 惰性计算和$符

惰性计算是一种被许多纯函数式语言采纳的计算策略，比如Haskell [H+92]。
这种计算策略有两个重要性质。
第一，给定表达式的计算会被推迟或中止，直到这个计算结果被需要。
第二，一旦一个被中止的表达式被求值，这个值会被记忆（即缓存），
所以下一次这个值被需要时，就能直接查找避免重复计算。

在严格计算语言中支持惰性计算（比如Standard ML）需要两个原语：
一个用来中止表达式的求值，另一个用来恢复被中止的表达式求值（并记忆求值的结果）。
这些原语经常被称为“延迟”和“推动”（这里用把force翻译为推动）。
举个例子，在新泽西州的Standard ML中提供了下面的原语来支持惰性求值

![2-1](/images/Functional-Data-Structures-2-1.png)

这些原语足以将本书中所有算法进行编码。
然而使用这些原语编程是很不方便的。
举个例子，要中止一个表达式<code>e</code>的求值，
要写`delay (fn () => e)`这么长一串。
根据用多少个空白符，前缀可以达到13-17个字符！
虽然当只有少数表达式要被延迟的时候，这是可以接受的，
但当有很多表达式需要被延迟的时候，这简直不能忍。

为了让延迟表达式语法不那么中，我们引入了$符来中止表达式求值。
对延迟表达式`e`。我们直接写<code>$e</code>就好了。
`$e`被称为一个suspension，拥有一个叫做`τ susp`的类型，
其中`τ`表示`e`的类型。
$符的作用域会尽可能向右延伸。
因此，`$f x`会被当做`$(f x)`而不是`($f) x`来解析,
`$x+y`会被当做`$(x+y)`而不是`($x)+y`。
要注意的是，`$e`本身也是一个表达式，所以也可以像`$$e`这样被延迟。
得到一个嵌套延迟类型`τ susp susp`

设`s`是一个延迟，类型是`τ susp`，那么`force s`计算并记忆`s`的内容，并返回类型为`τ`的结果。
然而，显式地用`force`来推动一个延迟也是很麻烦的。
特别地，这常常对模式匹配造成坏的影响，
会需要用`force`操作，把一个表达式拆成两个或更多个部分。
为了避免这样的问题，我们将$符与模式匹配一起引入。
在`$p`这种形式之前，匹配一个模式，会先推动延迟解决，然后再在`p`之前匹配这个结果。
有些时候，一个显式的`force`操作符仍然是相当有用的。然而现在，它能用$的形式来定义。

    fun force ($x) = x

为了对比两个符号，考虑一个标准的`take` 函数，功能是取得一个流的前n个元素。
流的定义如下：

![2-2](/images/Functional-Data-Structures-2-2.png)

使用`delay`和`force`写的`take`函数会长成这样

![2-3](/images/Functional-Data-Structures-2-3.png)


译者：如果你也和我一样没有见过ML语言，估计就懵逼了
戳这[wikipedia](https://en.wikipedia.org/wiki/Standard_ML)补补课吧，不谢。

相反，如果用上$运算符就可以写得更简洁些

![2-4](/images/Functional-Data-Structures-2-4.png)

实际上，他还可以写成这样诱人的形式

![2-5](/images/Functional-Data-Structures-2-5.png)

然而，这第三种实现和前两种并不相等。
它在绑定的时候就执行了第二个参数的运算，而不是等到求结果流的时候

$运算符语法和语义的正是定义在附录A中

##2.1 Streams

我们来写一个小数据流包，作为惰性求值和$运算符的一个拓展例子。
这个流也会在后面章节的一些数据结构中被用到。

流（也被称为惰性列表）与普通列表很相似，除了列表中的每一项都被系统地中止了。

![2-6](/images/Functional-Data-Structures-2-6.png)

一个简单的包含元素1,2,3的流可以写成

    $Cons (1, $Cons (2, $Cons (3, $Nil)))

把流和简单的类型为`α list susp`的被中止的列表放在一起比较是很有意义的。
后者的计算表现为一旦开始就一次性执行完毕，
另一方面，前者的要多次得推动stream执行恰好足够的计算，来得到外层的结果，并中止剩下的部分。
这种行为模式经常出现在，像这种包含嵌套中止的流等数据类型中。

为了更清楚的认识到这种行为的差异，考虑下自增函数`s ++ t`。在中止列表中，可能会被写成

    fun s ++ t = $(force s @ force t )

一旦开始，这个函数就推动两个参数执行然后拼接他们产生了整个结果。
因此，这个函数是一个整体。
在流中，`append`函数会被写成这样

![2-7](/images/Functional-Data-Structures-2-7.png)

一开始，这个函数推动`s`的第一个元素（通过匹配$模式）。
如果这个部分是Nil，那么结果的第一个元素就是`t`的第一个元素，所以函数函数推动`t`。
反之，函数用`s`的第一个元素来构造结果的第一个元素*--这里是关键--*。
这个中止最终会计算附加列表的剩余部分。
因此，这个函数是递增的。前面描述的`take`函数也是类似地递增的。

然而，我们来考虑下去掉流的前n个元素的函数

![2-8](/images/Functional-Data-Structures-2-8.png)

这个函数是一个整体，因为对`drop`的递归调用从来不会被延迟————
计算结果的第一个元素需要执行一整个`drop`函数。另一个整体的流函数是`reverse`。

![2-9](/images/Functional-Data-Structures-2-9.png)

这里对`reverse`的递归调用也没有被延迟过，
但要注意的是每次递归调用创建一个新的`$r`形式的中止。
这看起来可能会觉得`reverse`函数实际上没有一次性做完它的所有工作。
然而，像这种内容是明确的值的中止（也就是说完全由值和构造器组成，没有函数执行），
被称为平凡的。
一个合格的编译器会用早就记忆的形式来创建这些中止，
但就算编译器没有执行这种优化，平凡的中止总能在`O(1)`的时间内被解决。

虽然像`drop`和`reverse`这种一个整体的流函数很常见，
但是像`++`和`take`这种递增的函数在流中更有价值。
每一个中止处理一个微小但是重要的步骤，
所以为了效率最大化，懒惰执行只有在有必要的时候才应该被使用。
如果在惰性列表中只用到了整体的方法，那么应该使用简单的中止列表而不是流来实现。

![Figure 2.1](/images/Functional-Data-Structures-2-10.png)

Figure 2.1: A small streams package.

Figure 2.1用Standard ML总结了这些流函数。要注意流的类型定义是用Standard ML带类型类型的构造，
但是老版本的Standard ML不支持在签名中使用带类型的声明。
这个属性会在后面的版本中被支持，但如果编译器不允许，
那么简单的绕过方法是，删除流类型，并把每一个出现的`τ Stream`替换成`τ StreamCell susp`。
通过在签名中引入`StreamCell`数据类型，我们故意得选择暴露内部表示来支持流的模式。

##2.2 历史记录

惰性计算，由Wadsworth [Wad71] 首次提出，作为减少lambda运算的优化。
Vuillemin [Vui74] 随后表明，惰性计算在某些特定条件下，是最优的计算策略。
惰性计算的正式语义已经被广泛地研究 [Jos89, Lau93, OLT94, AFM+95]。

Streams Landin 提出了流 [Lan65]，但没有涉及记忆。
Friedman，Wise [FW76] ，Henderson和Morris [HM76]将它扩展成记忆的形式

Memoization Michie [Mic68]创造了`memoization`这个词来表示带参数-结果对缓存的函数参数。
（参数字段减少，当把中止当做无参函数来记忆中止）。
Hughes [Hug85]随后将Michie原始意义上的`memoization`投入函数式编程的使用。

构成惰性求值-延迟计算和记忆结果的算法在算法设计中有很长的历史，虽然不总是一起出现。
延迟潜在昂贵计算的操作（通常是删除），
在哈希表 [WV86], 优先队列 [ST86b, FT87]和搜索树 [DSST89]中表现良好。
另一方面，记忆是诸如动态规划[Bel57]和路径压缩 [HU73, TvL84]等技术的的基本原理。

惰性计算在CAML [W+90]早期版本的语法，和Standard ML很相似，
为惰性求值提供了类似$符的支持，而不是提供一个惰性构造器，
然而，CAML允许任何数据构造器被标记为惰性，
之后所有的构造器实例都会被惰性求值。
虽然这样比起$更加灵活，单页造成程序难以阅读。
用$符可以在语法上明显地分辨出哪些自表达式会被严格计算，哪些会被惰性计算，
但在CAML中，这个信息只能由类型声明指定。

