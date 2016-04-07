title: 中间件简述
tags: nodejs, connect middleware, 中间件
categories: nodejs, 自译自乐
---

# node connect 中间件简述
>译自[A short guide to Connect Middleware](http://stephensugden.com/middleware_guide/)

## 什么是Connect
参看其github的readme：
>Connect是node中一个可扩展并提供高性能插件（也被称为中间件）的HTTP服务框架。

更具体地说，connect封装了node的标准*http*模块并添加一些优秀的特性，*Server*对象*use*中间件堆栈(a stack of middleware)就是其中之一。

## 什么是中间件
简单地说，中间件是处理请求的函数。通过connect.createServer函数创建的server可以跟中间件堆栈关联在一起。当一个请求到来，它（请求）被传递到堆栈中的第一个中间件，一起传递的还有一个封装了ServerResponse的对象和一个next回调函数。每一个中间件都可以通过调用response对象中的方法对请求进行响应并且/或者调用next方法将请求传递到堆栈中的下一个中间件。一个无操作中间件的简例如下：
```javascript
	function uselessMiddleware(req, res, next){ next() }
```
中间件也可以出发一个错误，只要将错误作为next方法的第一个参数就可以了，如下：
```javascript
	function worseThanUselessMiddleware(req, res, next){
		next('this is an error msg')
	}
```
当一个中间件返回如上所示的错误，随后的所有中间件都将被跳过直到发现一个错误处理逻辑。（锚点*错误处理*）

往server的中间件堆栈添加中间件，使用use，如下所示：
```javascript
	connect = require('connect')
	stephen = connect.createServer()
	stephen.use(worseThanUselessMiddleware)
```
最后，添加中间件时可以指定路径前缀，只有请求匹配该路径前缀指定中间件才会对请求进行处理，如下：
```javascript
	connect = require('connect')
	bob = connect.createServer()
	bob.use('/attention', worseThanUselessMiddleware)
```

## 我能用它做什么
很多！常见的例子有日志、静态文件服务和错误处理。以上的三个标准中间件都已经内置在connect中，所以你可能不必亲自实现它们。另一个常见的中间件是将请求路由到控制器或者回调函数（欲了解更多，请参看TJ的express）。
实际上，你可以在任何你想要配置对所有请求的通用处理逻辑的地方使用中间件。例如，在我的[Lazorse](http://www.baidu.com)
项目中，请求路由和响应的渲染使用的是各自的中间件，因为它被设计与用于构建可能会使用不同渲染后端根据客户端请求header中的Accept属性。

## 示例

**基于URL的身份验证机制**
身份验证机制通常是由应用本身指定的，所以如下所示封装了connect.basicAuth和一个匹配身份验证url模式的列表
```javascript
	function authenticateUrls(urls /* basicAuth args*/){
		basicAuthArgs = Array.slice.call(arguments, 1) //arguments 下标为1的参数开始选取
		basicAuth = require('connect').basicAuth.apply(basicAuthArgs)
		function authenticate(req, res, next){
			for(var pattern in urls){
				if(req.path.match(pattern)){
					basicAuth(req, res, next)
					return
				}
			}

		next()
		}
		return authenticate
	}
```

**基于角色的认证**
```javascript
	// @urls - an object mapping patterns to lists of roles.
	// @roles - an object mapping usernames to lists of roles
	function authorizeUrls(urls, roles) {
		function authorize(req, res, next) {
			for (var pattern in urls) {
				if (req.path.match(pattern)) {
					for (var role in urls[pattern]) {
						if (users[req.remoteUser].indexOf(role) < 0) {
							next(new Error("unauthorized"))
							return
						}
					}
				}
			}
			next()
		}
		return authorize
	}
```
这些例子展示了中间件如何将请求逻辑分割成可替换的模块。如果我们决定用OAuth代替basicAuth，我们的认证模块包含的唯一的依赖仅仅是.remoteUser属性。

## 错误处理
另一个常见的横切关注度(cross-cutting concern)是错误处理。再次说明，connect已经内置优秀的错误处理机制，但通过客户来发现生产环境中的错误通常不是好的做法。
让我们使用[node_mailer](http://www.baidu.com)来实现一个简单的错误提醒中间件：
```javascript
	email= require('node_mailer')
	function emailErrorNotifier(generic_opts, escalate){
		function notifyViaEmail(err, req, res, next){
			if(err){
				var opts = {
					subject: 'Error: ' + err.constructor.name,
					body: err.stack,
				}
				opts.__proto__ = generic_opts
				email.send(opts, escalate)
			}
			next()
		}
	}
```
connect 检查中间件的参数数量，如果数量为4，则认为此中间件用于处理错误，也就是说之前的中间件处理过程成抛出的错误
会被当做错误处理中间件的第一个参数。

## 整合

下面是将上述的中间件都整合到了一起的app：
```javascript
	private_urls = {
		'^/attention': ['coworker', 'girlfriend'],
		'^/bank_balance': ['me'],
	}

	roles = {
		stephen :['me'],
		erin: ['girlfriend'],
		judd: ['coworker'],
		bob: ['coworker'],
	}

	passowrds = {
		me: 'doofus',
		erin: 'greatest',
		judd: 'daboss',
		bob: 'anachronistic discomBOBulation',
	}

	function authCallback(name, passowrd){
		return passowrds[name] === passowrd
	}

	stephen = require('connect').createServer();
	stephen.use(authenticateUrls(private_urls, authCallback))
	stephen.use(authorizeUrls(private_urls, roles))
	stephen.use('/attention', worseThanUselessMiddleware)
	stephen.use(emailErrorNotifier({ to: 'stephen@betsmartmedia.com'}))
	stephen.use('/bank_balance', function(req, res, next){
		res.end("Don't be Seb-ish")
	})
	stephen.use('/', function(req, res, next){
		res.end("I'm out of coffee")
	})

```

## 但我想用objects！
完全可行！传递给use方法一个包含*处理方法*的object，connect会以同样的方式调用此处理方法