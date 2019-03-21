#  axios源码学习



![结构&&流程：](https://github.com/cllee1214/blog/blob/master/images/axios.png)

注：学习源码使用的是打包之后的js文件。

## axios怎么来的：

在使用axios的时候一般是这样写的，如axios.get('/user?ID=12345').then(fun)。那么这个axios是怎么来的呢？

91行，可以看到axios是从createInstance返回的。

看看createInstance里面怎么写的（77行开始）：

1. 先是通过new Axios实例化了得到了一个上下文对象content。因此content这个对象上包含有Axios原型上的所有属性和方法（具体哪些方法看图）。
2. 这里的bind方法实际上是重新实现了原生的bind方法，即提供一个函数和一个对象，返回一个新函数instance，新函数的this指向这个对象。这里的作用就是把Axios.prototype.request的this指向第1步中的content。
3. 先把Axios.prototype的方法复制到instance上，且这些方法的this也是指向content（通过bind,看extend源码可知）。再把content的属性复制到instance上。

4. 返回instance（属性和方法具体看图）


可以看到，实际上这里的axios并不是一个对象，而是一个函数。使用的时候调用的是axios的静态方法（为什么这么设计？）


## 关于拦截器

拦截器的本质依旧是一个发布订阅模型。
1. InterceptorManager通过handlers存储拦截器订阅的回调函数。
2. use方法将这些函数加入到handlers中。需要注意的是handlers的结构为[{fulfilled: fun, rejected:fun}],是以成功失败结对的方式在一起的。
3. eject取消订阅

##  关于request方法

   构造函数Axios的原型上带了很多方法，如 'delete', 'get', 'head', 'options,'post', 'put', 'patch'。 实际上都是对request方法的封装（532行-551行），针对不同的请求方法处理不同的参数。所以这里只针对request方法来分析。

   2个大点：
   1. 加入了promise
   2. 这里涉及到axios的拦截器订阅的回调是如何加入promise的

   这里有一个回调函数链chain（实际上就是一个数组）（513行），结构有几个特点：一是函数都是以成对出现的，且前面一个是成功回调，后一个是失败回调（从关于拦截器第二点得来），如果没有的话则undifined。二是这个链分三部分,依次是请求拦截回调、发出请求、响应拦截回调。

  再看Promise.resolve(config)。由于config就只是一个普通对象，Promise.resolve在这里就是立即返还一个promise对象。实际上就是：
  `
   new Promise(function(resolve){
     resolve(config)
   })
   `

  想象中此promise对象后面会带有非常多的then方法，那么如何将这些拦截器回调作为参数传进去呢？

  (```)
  while (chain.length) {
	    promise = promise.then(chain.shift(), chain.shift());
	  };
  (```)

  需要了解的知识点是promise每次都会返回一个promise对象。那么这里的promise每次都会被上次的promis对象覆盖。
  这里每次都两个为一组从chain上拆下来作为参数传到then里面。为什么是两个？因为then的参数也是两个，分别对应fulfilled和rejected。且从上文分析chain也是这样的成对结构。
  整个过程就是：每次都返回一个promise，每次都在这个promise的then中加入回调，然后再返回一个promise,再在当前这个promise中加入回调。。循环这个过程，直到所有回调被添加完为止。

  看发出请求部分，即dispatchRequest：
  函数内部一大部分都是在做请求之前的参数处理，如处理请求地址、处理请求头、需要请求的数据参数等等。
  这里的重点在于adapter。
  简单的说，这个adapter才是真正处理请求地方。如果剔除其他一些细枝末节的东西，其实就是一个原生 promise + 原生ajax的 组合。即readyState为4且status为200执行resolve，其他的如error,timeout执行reject。


  