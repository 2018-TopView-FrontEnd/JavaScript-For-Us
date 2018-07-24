## 概述

在绝大多数情况下，函数的调用方式决定了`this`的值。在绝大多数情况下，函数的调用方式决定了`this`的值。`this`不能在执行期间被赋值，并且在每次函数被调用时`this`的值也可能会不同。
**`this`的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象**

**this 中的四种绑定方式 **

##  一 默认绑定
    console.log(this === window); 
    console.log(this.document === document); 
    this.a = 91;
    console.log(window.a); 

全局环境中，this默认绑定到window。

例子：

    var a = 0;
    function foo(){
        (function test(){
            console.log(this.a);
        })()
    };
    var obj = {
        a : 2,
        foo:foo
    }
    obj.foo();  


等价于上例

    var a = 0;
    var obj = {
        a : 2,
        foo:function(){
                function test(){
                    console.log(this.a);
                }
                test();
        }
    }
    obj.foo();//0
函数独立调用时，this默认绑定到window。

    (function () {
    	 console.log(this === window); // true
    })();

    function foo(){
        console.log(this === window);
    }
    foo();

## 二 隐式绑定

    function foo() {
        console.log( this.a )
      }

      var obj2 = { 
          a: 42,
          foo: foo
       }

       var obj1 = {
          a: 2,
          obj2: obj2
       }

      obj1.obj2.foo();  

结合上面的例子，this的指向可以总结为一下三种情况。

情况1：如果一个函数中有`this`，但是它没有被上一级的对象所调用，那么`this`指向的就是`window`，在js的严格版中`this`指向的不是`window`，而是`undefined`。

情况2：如果一个函数中有`this`，这个函数有被上一级的对象所调用，那么`this`指向的就是上一级的对象。

情况3：如果一个函数中有`this`，这个函数中包含多个对象，尽管这个函数是被最外层的对象所调用，`this`指向的也只是它上一级的对象。




##### 隐式丢失

    var o = {
    a:10,
    b: {
           a:12,
           fn:function(){
            console.log(this.a);  
            console.log(this);    
        }
      }
    }
    var j = o.b.fn;
    j();

**函数别名的情况**
赋值操作和调用不是同一个概念，上面说的 this是指向上一级调用它的对象，跟赋值操作没有上面关系。例子中虽然函数 fn 是被对象b 所引用，但是在将 fn 赋值给变量 j 的时候并没有执行所以最终指向的是 window。

    var a = 2;
    function foo() {
        console.log( this.a );
    }
    var o = { a: 3, foo: foo };
    var p = { a: 4 };
    o.foo(); 

    (p.foo = o.foo)();

**间接引用的情况**
等价于函数别名的情况。
如果要输出 4 ，该怎么做？

    var a = 2;
    function foo() {
        console.log( this.a );
    }
    var o = { a: 3, foo: foo };
    var p = { a: 4 };
    o.foo(); 
	p.foo = o.foo;
	p.foo();


## 三 显示绑定
就是给this指定值，如：new，call, apply, bind绑定等
## 1）call 和 apply 绑定

    function foo( something ) {
        console.log( this.a, something)
        return this.a + something
    }

    var obj = {
        a: 2
    }

    var bar = function() {
        return foo.apply( obj, arguments)
    }

    var b = bar(3); 
    console.log(b);
类型转换

    function bar() {
       console.log(Object.prototype.toString.call(this));
    }

    bar.call(7); 

使用 `call`和`apply`函数时要注意，如果传递给`this`的值不是一个对象，JS会尝试使用内部的`ToObject`操作将其转化为对象。因此，如果传递的值是一个原始值比如 7 ，那么就会使用相关构造函数将它转换为对象，所以原始值7会被转换为对象，像`new Number(7)`这样。

#### 2) bind 绑定

    var module = {
      x: 42,
      getX: function() {
        return this.x;
      }
    }

    var unboundGetX = module.getX;
    console.log(unboundGetX());          

    var boundGetX = unboundGetX.bind(module);
    console.log(boundGetX());
调用`f.bind(someObject)`会创建一个与`f`具有相同函数体和作用域的函数，但是在这个新函数中，`this`将永久地被绑定到了`bind`的第一个参数，无论这个函数是如何被调用的。返回由指定的`this`值和初始化参数改造的原函数拷贝

    function foo( something ) {
        console.log( this.a, something)
        return this.a + something
    }

    var obj = {
        a: 2
     }

    var bar = foo.bind(obj)

    var b = bar(3); 
    console.log(b); 
注意 bind指生效一次

## 3) new 绑定 
在传统面向类的语言中，使用`new`初始化类的时候会调用类中的构造函数，但是JS中`new`的机制实际上和面向类和语言完全不同。 

使用`new`来调用函数，或者说发生构造函数调用时，会自动执行下面的操作：

- 创建（或者说构造）一个全新的对象
- 这个新对象会被执行`[[Prototype]]`连接
- 这个新对象会绑定到函数调用的`this`

      function fn() {
          this.a = 2;
      }
      var test = new fn();
      console.log(test); 

**当 this 遇到 return 时**

在构造函数中使用`this`分为两种情况

     function fn()  
      {  
         this.user = 'zhong';  
         return undefined;       
      }
      var a = new fn;  
      console.log(a);   
函数当作构造函数使用且函数内没有返回值，或返回值是5种基本型（`Undefined`类型、`Null`类型、`Boolean`类型、`Number`类型、`String`类型）之一，指向实例后的对象

      function fn()  
      {  
          this.user = 'zhong';   
          return [1,2,3,4];
      }
      var a = new fn;  
      console.log(a);    

函数当作构造函数使用且有return值，返回是数组、对象等，this指向返回值

    function C2() {
      this.a = 1;
      return {
        a: 2
      };
    }
    o = new C2();
    console.log(o); 
   
   
my Note：   
- [JS中的this](https://github.com/chenyizhongx/Note/blob/master/javaScript/JS%E4%B8%AD%E7%9A%84this.md)
- [JS中的声明](https://github.com/chenyizhongx/Note/blob/master/javaScript/JS%E4%B8%AD%E7%9A%84%E5%A3%B0%E6%98%8E.md)



***
**Reference**
*   [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
*   [http://web.jobbole.com/85198/](http://web.jobbole.com/85198/)
*   [https://github.com/creeperyang/blog/issues/16](https://github.com/creeperyang/blog/issues/16)
*   [https://github.com/SimonZhangITer/MyBlog/issues/12](https://github.com/SimonZhangITer/MyBlog/issues/12)
* https://alexzhong22c.github.io/2017/08/07/js-this/ 

