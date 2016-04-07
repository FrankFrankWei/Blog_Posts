js笔记.md
知识点

## 一、
```javascript
var scope = 'global'
var func = function(){
	console.log(scope) //输出undefined
	var scope = 'f'
}

f()

```

这里没有输出global，而是undefined，这是为什么呢？这是JavaScript的一个特性
按照作用域搜索顺序，在console.log函数访问scope变量时，JavaScript会先搜索
函数f的作用域，恰巧在f作用域里面搜索到scope变量，所以上层作用域中的定义scope
就被屏蔽了，但执行console.log语句时，scope还没有被定义，或者说初始化，所以
得到的就是undefined了。


## 二、
函数作用域的嵌套关系是定义时决定的，而不是调用时决定的，也就是说，JavaScript的作用域是静态作用域，又叫词法作用域，这是因为作用域的嵌套关系可以在语法分析时
就确定，而不必等到运行时。举例如下：
var scope = 'top'

var func1 = function(){
	console.log(scope)
}

func1() //输出 top

var func2 = function(){
	var scope = 'f2'
	func1()
}

func2() //输出top

在这个例子中，通过func2调用的func1在查找scope定义时，找到的是父作用域中定义的
scope变量，而不是func2中定义的scope变量。这说明了作用域的嵌套关系不是在调用时确定的，而是在定义时确定的。

## 三、闭包
dao.js  main.js

假定我们想要给String增加一个deentityify方法，它的任务是寻找字符串中的HTML字符实体并替换为它们对应的字符。在一个对象中保存字符实体的名字和对应的字符是有意义的。但我们该在哪里保存该对象呢？我们可以把它放到一个全局变量中，但全局变量是魔鬼。我们可以把它定义在该函数本身，但是那有运行时的消耗，因为该函数在每次被执行的时候该字面量都会被求值一次。理想的方式是将其放入一个**闭包**，而且也许还能提供一个增加更多字符实体的扩展方法：
```javascript
//我们通过给Function.prototype增加方法来使得该方法对所有函数可用
Function.prototype.method = function(name, func){
	if(!this.prototype[name]){
		this.prototype[name] = func
	}
}

String.method('deentityify', function(){
	//字符实体表，它映射字符实体的名字和对应的字符
	var entity = {
		quot: '"',
		lt: '<',
		gt: '>'
	}

	return function(){
		// 这才是deentityify方法。它调用字符串的replace方法，
		// 查找‘&’开头和‘；’结束的子字符串。如果这些字符可以在字符实体表中找到
		// 那么就将该字符实体替换为映射表中的值。
		return this.replace(/$([^&;]+);/g,
		function(a, b){  // 关于这个函数的解释 https://msdn.microsoft.com/zh-cn/library/t0kbytzc(v=vs.94).aspx
				var r = entity[b]
				return typeof r === 'string' ? r : a
			})
	}

}())
```

## 四、原型继承


## 五、关于this
 javascript中共有四种调用模式：方法调用模式、函数调用模式、构造器调用模式和
 apply调用模式。

 1).方法调用模式
 当一个函数被保存为对象的一个属性时，我们称它为一个*方法*。当一个方法被调用时，
 this被绑定到该对象。如果一个调用表达式包含一个属性存取表达式（即一个"."点表达式或
 下标表达式），那么它被当作一个方法来调用。
 ```javascript
 // 创建myObject。它有一个value属性和一个increment方法
 // increment方法接受一个可选的参数。如果参数不是数字，使用默认值1.
 var myObject = {
 	value: 0;
 	increment: function(inc){
 		this.value += typeof inc === 'number' ? inc : 1
 	}
 }

 myObject.increment()
 document.writeln(myObject.value) // 1

 myObject.increment(2)
 document.writeln(myObject.value) // 3

 ```
 方法可以使用this去访问对象， 所以它能从对象中取值或修改该对象。this到对象的
 绑定**发生在调用的时候**。这个“超级”迟绑定（very late binding）使得函数可以对this高度
 复用。通过this可取得它们所属对象的上下文的方法称为**公共方法**

 2).函数调用模式
 当一个函数并非一个对象的属性时，那么它被当作一个函数来调用：
 var sum = add(3, 4) // sum的值为7
 当函数以此模式调用时，this被绑定到全局对象。这是语言设计上的一个错误，倘若语言设计正确，当内部函数被调用时，this应该仍然绑定到外部函数的this变量。这个设计错误的后果是方法不能利用内部函数来帮助它工作，因为内部函数的this被绑定了错误的值，所以不能共享该方法对对象的访问权。幸运的是，有一个很容易的解决方案：如果该方法定义一个变量并给它赋值为this，那么内部函数就可以通过那个变量访问到this。按照约定，我给那个变量命名为that：
 ```javascript
//给myObject增加一个double 方法
myObject.double = function(){
	var that = this  // 此时this指向myObject，that指向this，即that指向myObject对象
	var helper = function(){
		that.value = add(that.value, that.value) //调用时，this指向改变，绑定到全局对象,此时使用this就不会得到想要的结果，而that指向myObject对象，使用that能得到想要的结果
	}

	helper()
}

myObject.double()
document.writeln(myObject.getValue()) // 6 (上一步赋值为3了)
 ```
 