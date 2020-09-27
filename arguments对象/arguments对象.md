### arguments对象

题目

```js
function sidEffecting(arr) {
        arr[0] = arr[2];
        // console.log(arr);
    }

    function bar(a, b, c) {
        c = 10;
        sidEffecting(arguments);
        return a + b + c;
    }
    bar(1, 1, 1);
```

使用arguments引用函数的参数 。

**伪数组**：除了Array的**length**和**index**属性外没有其他任何属性

length 参数列表的长度

```js
var len=arguments.length;
typeof(arguments);//返回object
```

#### **1.转换成Array**

```js
var args = Array.prototype.slice.call(arguments);
var args = [].slice.call(arguments);


const args=Array.from(arguments);
ES6中
const args=[...arguments];
```

#### 2.定义连接字符串的函数

```js
// 定义连接字符串的函数
function myconcat(separator) {
    // args将参数转换成数组
    var args = [].slice.call(arguments, 1);
    // 使用join方法连接
    return args.join(separator);
}
//here*is*an*apple
var str = myconcat('*', 'here', 'is', 'an', 'apple');
```

#### 3.结合剩余参数、默认参数和解构赋值参数

1.在**严格模式**下，剩余参数、默认参数和解构赋值参数的存在不会改变 `arguments`对象的行为。

2.**非严格模式**下， **没有**包含剩余参数、默认参数和解构赋值参数，那么`arguments`对象中的值**会跟踪**	参数的值（**反之亦然** ）。

追踪 means **绑定**一块

```js
function func(a){
	arguments[0]=99;//更新arguments 同样更新了a
	console.log(a);
}
func(10);//99
```

```js
function func(a){
	a=99;	//更新a 同样更新了arguments[0]
	console.log(arguments[0]);
}
func(10);//99
```

3. 当非严格模式中的函数**有**包含剩余参数、默认参数和解构赋值参数，那么`arguments`对象中的值**不会**跟踪参数的值（反之亦然） 。

   没有绑定到一起

```js
function func(a = 55) {  //有默认参数
    arguments[0] = 99;
    console.log(a);
}
func(10); //10
```

