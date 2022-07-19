#迭代器——Iterator
##自定义迭代器
```javascript
function Range(from, to) {
    this.from = from;
    this.to = to;
    
    this[Symbol.iterator] = function () {
        return {
            current: this.from,
            last: this.to,
            
            next() {
                if (this.current <= this.last) {
                    return {
                        value: this.current++,
                        done: false
                    };
                } else {
                    return {
                        value: undefined,
                        done: true
                    }
                }
            }
        }
    }
}
//test
let range = new Range(1, 10);
for (let item of range) {
    console.log("item:", item);
}
```

##可迭代对象&伪数组转换为数组
###可迭代对象
```javascript
//上述栗子《自定义迭代器》
let range = new Range(1, 10);
let arr = Array.from(range);
console.log("arr:", arr);
console.log("isArray:", Array.isArray(arr));

//伪数组：有索引和 length 属性的对象
let arrObj = {
    0: "a",
    1: "b",
    length: 2
}
let arr = Array.from(arrObj);
```
> iterator 可以通过 [...] 方式转换为数组(array)
