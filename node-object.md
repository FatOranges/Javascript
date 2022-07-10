#对象
##检查空对象
```javascript
// 解决方案
function isEmpty(obj) {
    for (let key in obj) {
        return false;
    }
    return true;
}

// test 用例
isEmpty({name: "li"});//false
isEmpty({});//true
```
##深拷贝——对象
```javascript
//拷贝对象
let user = {
    name: "li",
    age: 15,
    size: {
        height: 500,
        width: 400,
        other: {
            address: "http"
        }
    }
}
//解决方案
function deepClone(obj) {
    let result = {};
    for (let key in obj) {
        if (typeof obj[key] === "object") {
            result[key] = deepClone(obj[key]);
        } else {
            result[key] = obj[key];
        }
    }
    return result;
}

//test
let result = deepClone(user);
user.size.other.address = "https";
console.log(result);
```
##深拷贝——lodash
###官方文档
https://lodash.com/docs#cloneDeep
###GitHub
https://github.com/lodash/lodash

##对象中的函数
###定义
```javascript
//方式一：
let user = {}
user.sayHi = function () {
    //...function body
}

//方式二：
let obj = {
    sayHi: function () {
        //...function body
    }
}

//方式三：
let abbrObj = {
    sayHi() {
        //...function body
    }
}
```

##方法中的 this
```javascript
//嵌套方法的情况下，如何改变 this 指向
//方式一：
let obj = {
    sayHi() {
        console.log(1111);
        let test = () => {
            console.log("name");
        }
        test();
    }
}
obj.sayHi();

//方式二：
let obj = {
    sayHi() {
        console.log(222);
        function test () {
            console.log("age");
        }
        test.call(obj);
    }
}
obj.sayHi();
```

#垃圾回收
##哪些状态不可回收
1. 当前执行的函数，和它的局部变量和参数。
2. 当前嵌套调用链上的其他函数、它们的局部变量和参数。
3. 全局变量。
4. 如果一个值可以通过引用链从根访问任何其他值。


#构造器 (constructor) 和操作符 new
##构造函数的约定
* 命名以大写字母开头。
* 只能由 new 操作符来执行。
```javascript
function User(name, age) {
    this.name = name;
    this.age = age;
}

let user = new User("zhang", 4);
console.log(user);
```

##构造器模式检测：new.target
> 再函数内部，我们可以使用 `new.target` 属性来检测它是否被使用 `new` 操作符进行调用的。
> 不使用 `new`，new.target 则为 undefined。
> 使用 `new`，new.target 则会改函数。
###使用场景
```javascript
//如果未正常使用 new 操作符创建对象，则强制创建。
function User(name, age) {
    if (!new.target) {
        new User(name, age);
    }
    this.name = name;
    this.age = age;
}

let user = User("li", 10);
console.log(user);
```

##构造器的 return
* 如果 `return` 返回的是一个对象，则返回这个对象，而不是 `this`。
* 如果 `return` 返回的是一个原始类型，则忽略。

##构造器中的方法定义
```javascript
function User(name, age) {
    //...
    this.sayHi = function (){
        //...function body
    }
}
```

#解构赋值——对象
```javascript
let cat = {
    name: "mao",
    age: 10,
    size: {
        height: 100,
        width: 100
    }
}
//重命名
let {name: a, age: b} = cat;
//解构后名称相同——简写
let {name, age} = cat;

//反向解构赋值
let name = "dog";
let age = 19;
let cat = {name, age};

//深层解构赋值
let {size: {height, width}} = cat;

//解构赋值——默认值
let {name, age, address = "http"} = cat;

//方法形参解构
function hello({name, age}) {
    console.log(name, age);
}

hello(cat);
```
