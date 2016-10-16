## JS 中 THIS 关键字详解

本文主要解释在JS里面this关键字的指向问题(**在浏览器环境下**)。

首先，必须搞清楚在JS里面，函数的几种调用方式:

+ 普通函数调用
+ 作为方法来调用
+ 作为构造函数来调用
+ 使用apply/call方法来调用
但是不管函数是按哪种方法来调用的，请记住一点：**谁调用这个函数或方法,this关键字就指向谁。**

接下来就分情况来讨论下这些不同的情况：

<h3>普通函数调用</h3>
````
javascript    function person(){
        this.name="xl";
        console.log(this);
        console.log(this.name);
    }

    person();  //输出  window  xl   
````
在这段代码中person函数作为普通函数调用，实际上person是作为全局对象window的一个方法来进行调用的,即window.person();
所以这个地方是window对象调用了person方法,那么person函数当中的this即指window,同时window还拥有了另外一个属性name,值为xl.
````
javascript    var name="xl";
    function person(){
        console.log(this.name);
    }
    person(); //输出 xl
````
同样这个地方person作为window的方法来调用，在代码的一开始定义了一个全局变量name，值为xl,它相当于window的一个属性,即window.name="xl",又因为在调用person的时候this是指向window的，因此这里会输出xl.

<h3>作为方法来调用</h3>

在上面的代码中，普通函数的调用即是作为window对象的方法进行调用。显然this关键字指向了window对象.

再来看下其他的形式
````
javascript    var name="XL";
    var person={
        name:"xl",
        showName:function(){
            console.log(this.name);
        }
    }
    person.showName();  //输出  xl
    //这里是person对象调用showName方法，很显然this关键字是指向person对象的，所以会输出name

    var showNameA=person.showName;
    showNameA();    //输出  XL
    //这里将person.showName方法赋给showNameA变量，此时showNameA变量相当于window对象的一个属性，因此showNameA()执行的时候相当于window.showNameA(),即window对象调用showNameA这个方法，所以this关键字指向window
````
再换种形式：
````
javascript    var personA={
        name:"xl",
        showName:function(){
            console.log(this.name);
        }
    }
    var personB={
        name:"XL",
        sayName:personA.showName
    }

    personB.sayName();  //输出 XL
    //虽然showName方法是在personA这个对象中定义，但是调用的时候却是在personB这个对象中调用，因此this对象指向
````
<h3>作为构造函数来调用</h3>

````
javascript    function  Person(name){
        this.name=name;
    }
    var personA=Person("xl");
    console.log(personA.name); // 输出  undefined
    console.log(window.name);//输出  xl
    //上面代码没有进行new操作，相当于window对象调用Person("xl")方法，那么this指向window对象，并进行赋值操作window.name="xl".

    var personB=new Person("xl");
    console.log(personB.name);// 输出 xl
    //这部分代码的解释见下
````
<h4>NEW操作符</h4>
````
javascript    //下面这段代码模拟了new操作符(实例化对象)的内部过程
    function person(name){
        var o={};
        o.__proto__=Person.prototype;  //原型继承
        Person.call(o,name);
        return o;
    }
    var personB=person("xl");

    console.log(personB.name);  // 输出  xl
````
+ 在person里面首先创建一个空对象o，将o的proto指向Person.prototype完成对原型的属性和方法的继承
+ Person.call(o,name)这里即函数Person作为apply/call调用(具体内容下方)，将Person对象里的this改为o，即完成了o.name=name操作
+ 返回对象o。

因此person("xl")返回了一个继承了Person.prototype对象上的属性和方法，以及拥有name属性为”xl”的对象，并将它赋给变量personB.
所以console.log(personB.name)会输出”xl”

<h3>CALL/APPLY方法的调用</h3>

在JS里函数也是对象，因此函数也有方法。从Function.prototype上继承到Function.prototype.call/Function.prototype.apply方法
call/apply方法最大的作用就是能改变this关键字的指向.

Obj.method.apply(AnotherObj,arguments);
````
javascript    var name="XL";
    var Person={
        name:"xl",
        showName:function(){
            console.log(this.name);
        }
    }
    Person.showName.call(); //输出 "XL"
    //这里call方法里面的第一个参数为空，默认指向window。
    //虽然showName方法定义在Person对象里面，但是使用call方法后，将showName方法里面的this指向了window。因此最后会输出"XL";
````
````
javascript    funtion FruitA(n1,n2){
        this.n1=n1;
        this.n2=n2;
        this.change=function(x,y){
            this.n1=x;
            this.n2=y;
        }
    }

    var fruitA=new FruitA("cheery","banana");
    var FruitB={
        n1:"apple",
        n2:"orange"
    };
    fruitA.change.call(FruitB,"pear","peach");

    console.log(FruitB.n1); //输出 pear
    console.log(FruitB.n2);// 输出 peach
````
FruitB调用fruitA的change方法，将fruitA中的this绑定到对象FruitB上。

另外几个需要注意的地方：
setTimeout/setInterval/匿名函数执行的时候，this默认指向window对象，除非手动改变this的指向。在《javascript高级程序设计》当中，写到：“超时调用的代码(setTimeout)都是在全局作用域中执行的，因此函数中的this的值，在非严格模式下是指向window对象，在严格模式下是指向undefined”。本文都是在非严格模式下的情况。
````
javascript    var name="XL";
    function Person(){
        this.name="xl";
        this.showName=function(){
            console.log(this.name);
        }
        setTimeout(this.showName,50);
    }
    var person=new Person(); //输出 "XL"

    //在setTimeout(this.showName,50)语句中，会延时执行this.showName方法
    //this.showName方法即构造函数Person()里面定义的方法。50ms后，执行this.showName方法，this.showName里面的this此时便指向了window对象。则会输出"XL";
````
修改上面的代码：
````
javascript    var name="XL";
    function Person(){
        this.name="xl";
        var that=this;
        this.showName=function(){
            console.log(that.name);
        }
        setTimeout(this.showName,50)
    }
    var person=new Person(); //输出 "xl"
    //这里在Person函数当中将this赋值给that，即让that保存Person对象，因此在setTimeout(this.showName,50)执行过程当中，console.log(that.name)即会输出Person对象的属性"xl"
````
<h3>匿名函数：</h3>

````
javascript    var name="XL";
    var person={
        name:"xl",
        showName:function(){
            console.log(this.name);
        }
        sayName:function(){
            function(callback){
                callback();
            }(this.showName)
        }
    }
    person.sayName();  //输出 XL
````
````
javascript    var name="XL";
    var person={
        name:"xl",
        showName:function(){
            console.log(this.name);
        }
        sayName:function(){
            var that=this;
            function(callback){
                callback();
            }(that.showName)
        }
    }
    person.sayName() ;  //输出  "xl"
    //匿名函数的执行同样在默认情况下this是指向window的，除非手动改变this的绑定对象
````
<h4>EVAL函数</h4>

该函数执行的时候，this绑定到当前作用域的对象上
````
javascript    var name="XL";
    var person={
        name:"xl",
        showName:function(){
            eval("console.log(this.name)");
        }
    }

    person.showName();  //输出  "xl"

    var a=person.showName;
    a();  //输出  "XL"
````
