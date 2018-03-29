>1. 字符串的遍历器接口

ES6 为字符串添加了遍历器接口（详见《Iterator》一章），使得字符串可以被for...of循环遍历

```
for (let i of 'foo') {
  console.log(i)
}
// "f"
// "o"
// "o"
```

除了遍历字符串，**这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点**

```
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```
上面代码中，字符串 text只有一个字符，但是**for循环会认为它包含两个字符（都不可打印）**，**而for...of循环会正确识别出这一个字符**
<br/>
<br/>
***
>2. at() 方法

ES5 对字符串对象提供 **charAt** 方法，**返回字符串给定位置的字符**。**该方法不能识别码点大于0xFFFF的字符**
```
'abc'.charAt(0) // "a"
'𠮷'.charAt(0) // "\uD842"
```
上面代码中的第二条语句，charAt方法期望返回的是用2个字节表示的字符，但汉字“𠮷”占用了4个字节，charAt(0)表示获取这4个字节中的前2个字节，很显然，这是无法正常显示的

**目前，有一个提案，提出字符串实例的at方法，可以识别 Unicode 编号大于0xFFFF的字符，返回正确的字符。**
```
'abc'.at(0) // "a"
'𠮷'.at(0) // "𠮷"
```
 ####notice:
 **请注意上面的at()方法只是提案，实际在浏览器控制台和node环境中是不能显示的。 会报错的！**
<br/>
<br/>
***
>3. includes(), startsWith(), endsWith()方法

传统上，JavaScript 只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6 又提供了三种新方法。

**includes()**：返回布尔值，表示是否找到了参数字符串。 
**startsWith()**：返回布尔值，表示参数字符串是否在原字符串的头部
**endsWith()**：返回布尔值，表示参数字符串是否在原字符串的尾部

```
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```
这三个方法都支持第二个参数，表示开始搜索的位置

```
let s = 'Hello world!';

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```
上面代码表示，**使用第二个参数n时，endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束**
<br/>
<br/>
***
>4. repeat() 方法

repeat方法返回一个新字符串，表示将原字符串重复n次
```
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```
参数如果是小数，会被取整，**向下取整**。

```
'na'.repeat(2.9) // "nana"
```
如果repeat的参数是负数或者Infinity，会报错:
```
  'na'.repeat(Infinity)
  // RangeError
 'na'.repeat(-1)
 // RangeError
```
但是，**如果参数是 0 到-1 之间的小数，则等同于 0，这是因为会先进行取整运算。0 到-1 之间的小数，取整以后等于-0，repeat视同为 0**

```
'na'.repeat(-0.9) // ""
```
参数NaN等同于0
```
'na'.repeat(NaN) // ""
```

如果repeat的参数是字符串，则会先转换成数字
```
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```
<br/>
<br/>
***

>5. 模板字符串

传统的 JavaScript 语言，输出模板通常是这样写的
```
$('#result').append(
  'There are <b>' + basket.count + '</b> ' +
  'items in your basket, ' +
  '<em>' + basket.onSale +
  '</em> are on sale!'
)
```
上面这种写法相当繁琐不方便，ES6 引入了模板字符串解决这个问题
```
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`)
```
模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中
```
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`)
```
**上面代码中，所有模板字符串的空格和换行，都是被保留的，比如<ul>标签前面会有一个换行。如果你不想要这个换行，可以使用trim方法消除它**

```
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim())
```
模板字符串中嵌入变量，需要将变量名写在**${}**之中
```
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 传统写法为
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```

大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性

```
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```
模板字符串之中还能调用函数
```
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

由于模板字符串的大括号内部，就是执行 JavaScript 代码，因此如果大括号内部是一个字符串，将会原样输出

```
`Hello ${'World'}`
// "Hello World"
```
<br/>
<br/>
***
