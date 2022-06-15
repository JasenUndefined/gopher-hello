# new Function()

> 持续创作，加速成长！这是我参与「掘金日新计划 · 6 月更文挑战」的第15天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468)

## 前言

在看一个源代码时发现有人写了这样的一段，一下子勾起我的好奇心，第一次遇到这样写法，且这样写法居然可以正常执行。其代码什么意思？为什么返回值是一个new Function()()?

```typescript
String.prototype.interpolate = function(){
	.....
	.....
	const names =[...]
	const values = [...]
	return new Function(...names, `return \`${this}\`;`)(...values);
}
```



## 函数定义

函数就是一个代码块。只需要定义一次，但可能被执行或调用任意次。

在平常使用频率比较多的定义函数方式就是函数声明式定义方式，就是在 JavaScript 中是常常创建一个函数的使用方式：

```javascript
function func_name(arg1,arg2...){
		// body :to do something
}
```

有时候，可能使用函数表达式的方式定义函数。

```javascript
let getUserInfo = function(id) {
	...
}
```

##  new Function 定义函数

在 ES6 的模板字符串的案例中有一个创建函数的语法是这样的：

```tsx
let str = 'return ' + '`Hello ${name}!`';
let func = new Function('name', str);
func('Jack') // "Hello Jack!"
```

### 语法

```tsx
let func = new Function ([arg1[, arg2[, ...argN]],] functionBody)
```

其中arg1,arg2直到argN就是我们需要传递的形参，可以有任意个，最后一个function_body就是我们希望函数执行的函数体，这里函数体必须放在最后。注意一点，参数和函数体都必须用字符串的形式写入。

**new Function()**的方式和其他的方式不一样，函数是由在运行时传入的字符串创建的。函数声明式和函数表达式的声明都是在脚本中编写功能代码。

### new Function() 作用域

通常，函数将它所创建的位置记录在特殊属性[[Environment]]中。 它引用了创建地点的词法环境。但是当使用new Function()创建函数时，其[[Environment]]不是引用当前的词法环境，而是引用**全局环境**。

通过一个 Demo 来讲解：

```javascript
function getUser(){
  let name ="tommy"
  let func = new Function('alert(value)');
  return func;
}

getUser()() // 这样执行时将报错，value 未定义

// 如果定义 value 后,则可以政策执行
let value = “我是全局 Tom”;

```







参考资料：

- https://blog.csdn.net/qq_34629352/article/details/119848863

