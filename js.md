# js装b大法

### 1. 获取一个随机布尔值 (true/false)
这个函数使用 Math.random() 方法返回一个布尔值（true 或 false）。Math.random 将在 0 和 1 之间创建一个随机数，之后我们检查它是否高于或低于 0.5。这意味着得到真或假的几率是 50%/50%。

```
const randomBoolean = () => Math.random() >= 0.5;
console.log(randomBoolean());
```

### 2. 检查日期是否为工作日
使用这个方法，你就可以检查函数参数是工作日还是周末。

```
const isWeekday = (date) => date.getDay() % 6 !== 0;
console.log(isWeekday(new Date(2021, 0, 11)));
```


### 3. 反转字符串
有几种不同的方法来反转一个字符串。以下代码是最简单的方式之一。

```
const reverse = str => str.split('').reverse().join('');
reverse('hello world');     
```

### 4. 检查当前 Tab 页是否在前台

```
const isBrowserTabInView = () => document.hidden;
isBrowserTabInView();
```

### 5. 检查数字是否为奇数
最简单的方式是通过使用模数运算符（%）来解决。如果你对它不太熟悉，这里是 Stack Overflow 上的一个很好的图解。

```
const isEven = num => num % 2 === 0;
console.log(isEven(2));
```

### 6. 从日期中获取时间
通过使用 toTimeString() 方法，在正确的位置对字符串进行切片，我们可以从提供的日期中获取时间或者当前时间。

```
const timeFromDate = date => date.toTimeString().slice(0, 8);
console.log(timeFromDate(new Date(2021, 0, 10, 17, 30, 0))); 
```

### 7. 保留小数点（非四舍五入）
使用 Math.pow() 方法，我们可以将一个数字截断到某个小数点。

```
const toFixed = (n, fixed) => ~~(Math.pow(10, fixed) * n) / Math.pow(10, fixed);
// Examples
toFixed(25.198726354, 1);       // 25.1
toFixed(25.198726354, 2);       // 25.19
toFixed(25.198726354, 3);       // 25.198
toFixed(25.198726354, 4);       // 25.1987
toFixed(25.198726354, 5);       // 25.19872
toFixed(25.198726354, 6);       // 25.198726
```

### 8. 检查元素当前是否为聚焦状态
我们可以使用 document.activeElement 属性检查一个元素当前是否处于聚焦状态。

```
const elementIsInFocus = (el) => (el === document.activeElement);
elementIsInFocus(anyElement)
```

### 9. 检查浏览器是否支持触摸事件

```
const touchSupported = () => {
  ('ontouchstart' in window || window.DocumentTouch && document instanceof window.DocumentTouch);
}
console.log(touchSupported());
```

### 10. 检查当前用户是否为苹果设备

我们可以使用 navigator.platform来检查当前用户是否为苹果设备。

```
const isAppleDevice = /Mac|iPod|iPhone|iPad/.test(navigator.platform);
console.log(isAppleDevice);
```

### 11. 滚动到页面顶部
window.scrollTo() 方法会取一个 x 和 y 坐标来进行滚动。如果我们将这些坐标设置为零，就可以滚动到页面的顶部。

```
const goToTop = () => window.scrollTo(0, 0);
goToTop();
```

### 12. 获取所有参数平均值
我们可以使用 reduce 方法来获得函数参数的平均值。

```
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
average(1, 2, 3, 4);
```

