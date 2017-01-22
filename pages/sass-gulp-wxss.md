#  记用gulp将scss直接生成wxss

## 为什么会有这篇文章

* 由于微信小程序只能使用wxss后缀的样式文件
* 微信小程序的编辑器不好用 写样式嵌套麻烦（也许是sass后遗症）
* 也再次学习一下gulp

### 那么用sass来生成wxss可以吗？是不是就解决了嵌套问题？实践下来，遇到如下问题：


## 一、sass默认是编译成css文件的，那么如何直接把后缀名改成wxss呢?
 <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以安装gulp相关插件：<a href="https://www.npmjs.com/package/gulp-rename">gulp-rename</a>，将后缀名改为wxss</p>

## 二、不要将sass的node_modules相关代码放到小程序目录下。

 <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;就目前而言，小程序还不支持外部的node模块。如果将node模块放入小程序目录下可能会使得小程序编辑器报错</p>


## 三、gulp相关的
  
 <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在gulp编译过程中，我们可能会发现速度很慢。因为gulp是默认把所有文件都去编译一遍。这样肯定不行，于是百度之，找到了<a href="https://www.npmjs.com/package/gulp-rename">gulp-changed</a>，顾名思义，change嘛，改变嘛。即我们可以做到只编译发生变动的文件。</p>
 <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果这样的话我们只是做了一半工作，实际上一个文件可能是js scss或者其他什么文件。我们通过文件的扩展名筛选是否编译这个文件</p>


## 四、可以用批处理将css改成wxss
 <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 如 ren *.css *.wxss </p>

</br>
</br>

# 附代码

 ```
var gulp = require("gulp");
var sass = require("gulp-sass");
var rename = require("gulp-rename");
var changed = require("gulp-changed");

//定义 sass文件源地址 wxss文件地址
var origin = "sass/**/*.scss";
var target = "../wxAppOnline1905/public/static/wxss";

//编译sass任务 =》 gulp sass
gulp.task("sass", function(){
    gulp.src(origin)
     //筛选个已经更改并且后缀是scss的文件
    .pipe(changed(target, {extension: "scss"}))
    //展开编译的文件
    .pipe(sass({outputStyle: "expanded"}))
    //后缀名改为wxss
    .pipe(rename(function(path){
        path.extname = ".wxss";
    }))
    .pipe(gulp.dest(target))
})



//默认任务 => gulp
gulp.task("default", function(){
    gulp.run("sass");
})

//监听sass文件变化，执行编译sass任务 => gulp watch:sass
gulp.task("watch:sass",function(){
    gulp.watch(origin, ["sass"]);
})```
