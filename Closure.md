## javascript闭包的理解
javascript闭包是javascript的难点，很多人对js闭包不是很理解，我对js闭包一开始也是云里雾里，我刚刚进兴安得力的时候，做的转正试题中就有一个对闭包理解的题目。如何理解javascript的闭包呢？下面我们一起来学习一下：

闭包的含义和理解

通俗地讲，JavaScript 中每个的函数都是一个闭包，但通常意义上嵌套的函数更能够体现出闭包的特性，请看下面这个例子：
````
var   generateClosure   =   function()   {
    var   count   =   0;
    var   get   =   function()   {
    count   ++;
    return   count;
         };
    return   get;
};
var   counter   =   generateClosure();
console.log(counter());   //   输出   1
console.log(counter());   //   输出   2
console.log(counter());   //   输出   3
````
这段代码中，generateClosure() 函数中有一个局部变量count，初值为 0。还有一个叫做 get 的函数，get 将其父作用域，也就是 generateClosure() 函数中的 count变量增加 1，并返回 count 的值。generateClosure() 的返回值是 get 函数。在外部我们通过 counter变量调用了 generateClosure() 函数并获取了它的返回值，也就是 get 函数，接下来反复调用几次 counter()，我们发现每次返回的值都递增了1。

让我们看看上面的例子有什么特点，按照通常命令式编程思维的理解，count 是generateClosure 函数内部的变量，它的生命周期就是 generateClosure 被调用的时期，当 generateClosure 从调用栈中返回时，count 变量申请的空间也就被释放。问题是，generateClosure() 调用结束后，counter() 却引用了“已经释放了的” count变量，而且非但没有出错，反而每次调用 counter() 时还修改并返回了 count。这是怎么回事呢？

这正是所谓闭包的特性。当一个函数返回它内部定义的一个函数时，就产生了一个闭包，闭包不但包括被返回的函数，还包括这个函数的定义环境。上面例子中，当函数generateClosure() 的内部函数 get 被一个外部变量 counter 引用时，counter 和generateClosure()的局部变量就是一个闭包。如果还不够清晰，下面这个例子可以帮助你理解：
````
var   generateClosure   =   function()   {
    var   count   =   0;
    var   get   =   function()   {
        count   ++;
        return   count;
    };
    return   get;
};

var   counter1   =   generateClosure();
var   counter2   =   generateClosure();
console.log(counter1());   //   输出   1
console.log(counter2());   //   输出   1
console.log(counter1());   //   输出   2
console.log(counter1());   //   输出   3
console.log(counter2());   //   输出   2
````
上面这个例子解释了闭包是如何产生的： counter1和counter2分别调用了generate-Closure() 函数，生成了两个闭包的实例，它们内部引用的 count 变量分别属于各自的运行环境。我们可以理解为，在 generateClosure() 返回 get 函数时，私下将 get 可能引用到的 generateClosure() 函数的内部变量（也就是 count 变量）也返回了，并在内存中生成了一个副本，之后 generateClosure() 返回的函数的两个实例 counter1和 counter2 就是相互独立的了。

闭包的用途

1. 嵌套的回调函数

闭包有两个主要用途，一是实现嵌套的回调函数，二是隐藏对象的细节。让我们先看下面这段代码示例，了解嵌套的回调函数。如下代码是在 Node.js 中使用 MongoDB 实现一个简单的增加用户的功能（可以说每一个nodejs的高手，通常也是js的高手）：
````
exports.add_user   =   function(user_info,   callback)   {
    var   uid   =   parseInt(user_info['uid']);
    mongodb.open(function(err,   db)   {
        if   (err)   {callback(err);   return;}
        db.collection('users',   function(err,   collection)   {
            if   (err)   {callback(err);   return;}
            collection.ensureIndex("uid",   function(err)   {
                if   (err)   {callback(err);   return;}
                collection.ensureIndex("username",   function(err)   {
                    if   (err)   {callback(err);   return;}
                    collection.findOne({uid:   uid},   function(err)   {
                        if   (err)   {callback(err);   return;}
                        if   (doc)   {
                            callback('occupied');
                        }   else   {
                            var   user   =   {
                                uid:   uid,
                                user:   user_info,
                            };
                            collection.insert(user,   function(err)   {
                                callback(err);
                                           }）
                                     }
                            }）
                        }）
                   }）
              }）
        }）
}
````
如果你对 Node.js 或 MongoDB 不熟悉，没关系，不需要去理解细节，只要看清楚大概的逻辑即可。这段代码中用到了闭包的层层嵌套，每一层的嵌套都是一个回调函数。回调函数不会立即执行，而是等待相应请求处理完后由请求的函数回调。我们可以看到，在嵌套的每一层中都有对 callback 的引用，而且最里层还用到了外层定义的 uid 变量。由于闭包机制的存在，即使外层函数已经执行完毕，其作用域内申请的变量也不会释放，因为里层的函数还有可能引用到这些变量，这样就完美地实现了嵌套的异步回调。

2. 实现私有成员

我们知道，JavaScript 的对象没有私有属性，也就是说对象的每一个属性都是曝露给外部的。这样可能会有安全隐患，譬如对象的使用者直接修改了某个属性，导致对象内部数据的一致性受到破坏等。JavaScript通过约定在所有私有属性前加上下划线（例如_myPrivateProp），表示这个属性是私有的，外部对象不应该直接读写它。但这只是个非正式的约定，假设对象的使用者不这么做，有没有更严格的机制呢？答案是有的，通过闭包可以实现。让我们再看看前面那个例子：
````
var   generateClosure   =   function()   {
        var   count   =   0;
        var   get   =   function()   {
            count   ++;
            return   count;
        };
        return   get;
    };

    var   counter   =   generateClosure();
    console.log(counter());   //   输出   1
    console.log(counter());   //   输出   2
    console.log(counter());   //   输出   3
````  
我们可以看到，只有调用 counter() 才能访问到闭包内的 count 变量，并按照规则对其增加1，除此之外决无可能用其他方式找到 count 变量。受到这个简单例子的启发，我们可以把一个对象用闭包封装起来，只返回一个“访问器”的对象，即可实现对细节隐藏
