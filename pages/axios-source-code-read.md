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