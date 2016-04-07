#nodejs中exports和module.exports理解。

　　通常，Node.js应用程序会由多个模块组成，模块是Node.js应用程序的基本组成部分，一个Node.js文件就是一个模块。
　　Node.js提供了module.exports和require两个对象来支持模块之间的互动：module.exports用于定义模块对外暴露的部分，**require用于获取指定模块的module.exports对象。
　　exports是一个初始引用与module.exports相同的变量。**
　　从加粗的文字中我们可以得出一个结论：
　　我们分两种情况来讨论：
### 一、模块暴露方法
> ***初始时** exports = module.exports = {}*
```javascrpit
//methods.js
function SayHello(name){
	console.log('hello %s', name)
}

function SayGoodbye(name){
	console.log('bye %s', name)
}

module.exports.Hello = SayHello //exports.Hello = SayHello 两种方式都可以
module.exports.Bye = SayGoodbye //exports.Bye = SayGoodbye 两种方式都可以
```

>***此时** exports = module.exports = { Hello: SayHello, Bye: SayGoodBye}*
*SayHello, SayGoodBye是两个**方法的引用***

　　接下来完成一个调用methods.js的模块:
```js
// invoke.js
var methods = require('./methods')
methods.Hello('Frank')
methods.Bye('Frank')
```

### 二、模块暴露对象


