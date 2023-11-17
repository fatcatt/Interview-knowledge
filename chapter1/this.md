# Section1.2

#### 以下文章对 this 的描述非常详尽，对默认绑定、显示绑定、隐式绑定、new 绑定，以及可能遇到的特殊情况，都做了详尽的举例，直接上文章，就不一一列举了。👇

-   推荐好文：[一篇文章肝爆 this](https://juejin.cn/post/6844904083707396109#heading-45)。

#### 特别提醒：

-   隐式绑定中的隐式丢失问题

    -   ```
        var obj1 = {
            a: 1
        }
        var obj2 = {
        a: 2,
        foo1: function () {
            console.log(this.a)
        },
        foo2: function () {
            setTimeout(function () {
            console.log(this)
            console.log(this.a)
            }, 0)
        }
        }
        var a = 3

        obj2.foo1()
        obj2.foo2()
        ```

        > 对于 setTimeout 中的函数，这里存在隐式绑定的隐式丢失，也就是当我们将函数作为参数传递时会被隐式赋值，回调函数丢失 this 绑定，因此这时候 setTimeout 中的函数内的 this 是指向 window 的。所以最终的结果是：2 window(...) 3

-   call、apply 的位置问题

    -   ```
            function foo () {
                console.log(this.a)
            }
            var obj = { a: 1 }
            var a = 2

            foo()
            foo.call(obj)
            foo().call(obj)
        ```

        > foo.call(obj)是显示绑定了 this，改变了 foo 内 this 的指向，所以 obj.a 是 1；foo().call(obj)是在执行 foo()后又.call(obj)，所以会先打印执行结果 2， 再打印 Uncaught TypeError: Cannot read property 'call' of undefined, 因为 foo()函数的返回值是 undefined
