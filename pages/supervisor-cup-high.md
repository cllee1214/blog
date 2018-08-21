#  记解决supervisor cup占用过高问题

一个官网项目，目录结构是这样的：  

![目录结构]('https://github.com/cllee1214/blog/blob/master/images/1.png)

启动项目:  `supervisor sever.js`

发现就node占用cpu接近97%！  
supervisor有监听文件的作用，回头看目录结构，发现node_modules在入口文件的同级下。尝试忽略此目录：  

`supervisor  -i node_modules,views,source server.js`

然后再启动项目，cpu占用率就正常了。  


----------------------------

参考：  ['supervisor github地址'](https://github.com/petruisfan/node-supervisor)
