##　关于javascript中apply()和call()方法的区别
ref：http://www.popo4j.com/article/the-differences-of-apply-and-call.html

　　如果没接触过动态语言,以编译型语言的思维方式去理解javaScript将会有种神奇而怪异的感觉,因为意识上往往不可能的事偏偏就发生了,甚至觉得不可理喻.如果在学JavaScript这自由而变幻无穷的语言过程中遇到这种感觉,那么就从现在形始,请放下的您的”偏见”,因为这对您来说绝对是一片新大陆,让JavaScrip

好,言归正传,先理解JavaScrtipt动态变换运行时上下文特性,这种特性主要就体现在apply, call两个方法的运用上.

**区分apply,call就一句话,**
````
　　foo.call(this, arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)
````
 

call, apply都属于Function.prototype的一个方法,它是JavaScript引擎内在实现的,因为属于Function.prototype,所以每个Function对象实例,也就是每个方法都有call, apply属性.既然作为方法的属性,那它们的使用就当然是针对方法的了.这两个方法是容易混淆的,因为它们的作用一样,只是使用方式不同.

**相同点:两个方法产生的作用是完全一样的**

**不同点:方法传递的参数不同**

那什么是方法产生的作用,方法传递的参数是什么呢?

我们就上面的foo.call(this, arg1, arg2, arg3)展开分析.

foo是一个方法,this是方法执行时上下文相关对象,arg1, arg2, arg3是传给foo方法的参数.这里所谓的方法执行时上下文相关对象, 如果有面向对象的编程基础,那很好理解,就是在类实例化后对象中的this.

**在JavaScript中,代码总是有一个上下文对象,代码处理该对象之内. 上下文对象是通过this变量来体现的, 这个this变量永远指向当前代码所处的对象中.**


对象的方法可以任意指派,而对象本身一直都是没有这方法的,注意是指派,通俗点就是,方法是借给另一个对象的调用去完成任务,原理上是方法执行时上下文对象改变了.

所以 b.setMessage.call(a, “a的消息”); 就等于用a作执行时上下文对象调用b对象的setMessage方法,而这过程中与b一点关系都没有, 作用等效于a.setMessage( “a的消息”);

因为apply与call产生的作用是一样的,可以说

**call, apply作用就是借用别人的方法来调用,就像调用自己的一样.**

好,理解了call, apply相同处—–作用后,再来看看它们的区别,看过上面例子,相信您大概知道了.

从 b.setMessage.call(a, “a的消息”) 等效于 a.setMessage( “a的消息”) 可以看出, “a的消息”在call中作为一个参数传递,

那么在apply中是怎么表示的呢,直接解释说不清楚,apply要结合应用场景才一目了然.我们来设计一个应用场景:
 

````
function print(a, b, c, d){

alert(a + b + c + d);

}

function example(a, b , c , d){

//用call方式借用print,参数显式打散传递

print.call(this, a, b, c, d);

//用apply方式借用print, 参数作为一个数组传递,

//这里直接用JavaScript方法内本身有的arguments数组
09
print.apply(this, arguments);

//或者封装成数组

print.apply(this, [a, b, c, d]);

}

//下面将显示”背光脚本”

example(”背” , “光” , “脚”, “本”);
````
在这场景中, example方法内,a, b, c, d作为方法传递的参数, 方法分别运用了apply, call去借print方法来调用,

最后一句由于直接调用example方法, 所以在该方法中的上下文对象this就是window对象.

所以,call, apply方法它们除了第一个参数,即执行时上下文对象相同外,call方法的其它参数将依次传递给借用的方法作参数,而apply就两个参数,第二个参数为一个数组传递.所以可以说成

**call, apply方法区别是,从第二个参数起, call方法参数将依次传递给借用的方法作参数, 而apply直接将这些参数放到一个数组中再传递, 最后借用方法的参数列表是一样的.**

 

应用场景:

当参数明确时可用call, 当参数不明确时可用apply给合arguments
 

````
//例

print.call(window, “背” , “光” , “脚”, “本”);

//foo参数可能为多个

function foo(){

print.apply(window, arguments);

}
````
