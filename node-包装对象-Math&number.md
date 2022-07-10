#number
##编写数字的语法糖
> let billion = 1_000_000_000;
> 
> let billion1 = 1e9;
> 
> 1.23e6 === 1.23 * 1000000;
> 
> 0.000001 === 1e-6;

##数字转进制
```javascript
//Number.toString(base);
let num = 255;
num.toString(2);//转换为 2 进制。
num.toString(16);//转换为 16 进制。
```
> 注：base 的范围可以从 2 到 36。默认情况下是 10

#Math
##Math.floor
> 向下取整
```javascript
Math.floor(1.1);//1
Math.floor(1.9);//1
Math.floor(-1.2);//-2
```
##Math.ceil
> 向上取整
```javascript
Math.ceil(1.1);//2
Math.ceil(1.9);//2
Math.ceil(-1.3);//-1
```
##Math.round
> 四舍五入
```javascript
Math.round(1.4);//1
Math.round(1.5);//2
Math.round(-1.2);//-1
Math.round(-1.5);//-2
```
##Math.trunc
> 直接舍弃小数部分
```javascript
Math.trunc(1.1);//1
Math.trunc(1.9);//1
Math.trunc(-1.4);//-1
```
##Math.toFixed
> 将数字舍入到小数点后 n 位，并以字符串形式返回结果。
> 
> 位数不够用 0 来凑。
> 
> 四舍五入。
```javascript
let num = 12.34;
num.toFixed(1);//"12.3"
num.toFixed(5);//"12.34000"
//转换为数字
+num.toFixed(5);//12.34000
```

#isNaN
> 是否为 NaN
```javascript
isNaN(NaN);//true
isNaN("str");//true
isNaN(14);//false
```

#isFinite
> 是否是个有限数。
> 
> NaN/Infinity/-Infinity 都会返回 false。
```javascript
isFinite("15");//true
isFinite("str");//false
isFinite(Infinity)//false
```

#小知识
> console.log(9999999999999999);//10000000000000000
>
> console.log(0 === -0);//true
>
> NaN === NaN; //false
>
> Number();//0
>
> Number("");//0
>
> Number("     ");//0

#判断一个值是否为 number 类型
##isNaN
```javascript
function checkInp() {
    let x = prompt("enter a value");
    if (isNaN(x)) {
        /*由于 isNaN 之前会将传入的参数强转为 number 类型，当用户输入的为 null,"","    " 会转换为 0
        * 离预期就会出现偏差，此方法不可取
        * */ 
        console.log("请输入数字");
        return false;
    }
}
```
##正则
```javascript
function checkInp() {
    let x = prompt("enter a value:");
    let regex = /^[0-9]+$/;
    if (!x.match(regex)) {
        console.log("请输入数字");
        return false;
    }
}
```
##jQuery☆☆☆☆☆
```javascript
function checkInp() {
    let x = prompt("enter a value:");
    return !isNaN(parseFloat(x)) && isFinite(x);
}
```
#随机一个区间数
```javascript
function getRandom(min, max) {
    return min + Math.random() * (max - min);
}

function getRandomInteger(min, max) {
    min = Number.isInteger(min) ? min : Math.ceil(min);
    max = Number.isInteger(max) ? max : Math.floor(max);
    return Math.floor(getRandom(min, max));
}
```
