# 介绍



## vkitty是什么？

vkitty是一种前端构建工具。取名自hello kitty，其希望能够打造成一个简单、灵活的前端构建工具。

与其他工具相比，他语义化，更加灵活，更加高效，扩展性高，可适用于小到组件开发，大到商城页面构建，可配合现在市场上主流的库和框架，如vue、react、jquery。



## 特点

* 简单-没有大量而复杂的配置，使用结构类似gulp，并语义化完成代码架构及打包，对项目管理非常友善，方便修改和交接。
* 灵活-通过独有loaders解决方案，可以构建出各种开发者期望的结果。
* 强大-支持自定义loaders，强力支持gulp海量插件，支持预变量处理，拥有非常方便的钩子loader。
* 快速-vkitty使用反向编译机制，简单的说就是只会编译修改了的文件和被其影响到的文件，所以大型项目开发也能快速编译。

对比起webpack

优点：学习难度低，灵活，配置简单，编译速度快，文件结构会更加合理，打包的代码体积小，海量插件。

缺点：暂不支持es6 to es5(真心不理解用es6去写es5代码)，错误提示不够友好，另外就是没经过时间考验，可能会出现不稳定情况。

题外：如果写一个小小的组件都要用到webpack，那将是如何的酸爽/(ㄒoㄒ)/~~



## 安装

```
npm install vkitty --save
```



## 起步

index.html

```html
<body>
	_include('./index.js?_ac=jstag')
</body>
```

index.js

```javascript
var css = _include('./index.less?_ac=less',{"color":"red"});
var test = require('./index.js');
console.log(css);
```

index.less

```less
body{
  color: @@color@@;
  background:url(_include('./index.png?_ac=cdn','#cdnurl#'));
}
```

gulpfile.js

```javascript
const gulp = require('gulp');
const kitty = require('vkitty');
const loaders = kitty.loaders;

gulp.task('dev',function(){
    kitty.watch('./pages/*/*.html')
        .pipe(kitty.dest('./build/pages'))
        .pipe(kitty.cdnDest('./build/static'))
});
```



## 默认配置

```javascript
var config = {
    tag:'_include',//引入tag
    baseDir:process.cwd(),//项目工作目录
    varTag: {
        open: '@@',//变量开始符
        close: "@@"//变量结束符
    }
};
```

其挂载在kitty.config下面，大部分情况下，配置是不需要改动的。



# _include使用方法

格式：

```html
_include('filename?[_ac=loaders]'[,options])
```

实例：

```html
_include('./index.less?_ac=less',{"color":"red"});
```



* filename - 文件名称，可以是绝对路径和相对路径。
* loaders - 文件编译模块名，用`-`分隔。
* options - 选项，设置加载的预定义变量对象。


## 预变量

```json
var config
    varTag: {
        open: '@@',//变量开始符
        close: "@@"//变量结束符
    }
};
```

每个_include或require后可配置options。

```html
index.less文件
body{
  color:'@@color@@'
}

引入方式
var css = _include('./index.less?_ac=less',{"color":"red"});
```

*注意options需要标准的json格式。预变量不支持逻辑判断，如果需要逻辑判断，应该在钩子函数里面完成，以保持业务代码的整洁。*



## 方法

vkitty使用流处理方式，使用方式类似gulp。

## kitty.src

格式，请参考gulp.src。

```javascript
kitty.src(['./pages/**/*.html', '!./pages/vendor/*.html']) 
```
*我们只要保证入口文件引入就可以了。*


## kitty.watch

和gulp.watch不同，gulp.watch是通过回调来处理后续任务，这里是直接生成可读流，可直接参考gulp.src用法。

```
kitty.watch(['./pages/**/*.html', '!./pages/vendor/*.html']) 
```

推荐开发时候用kitty.watch，上线时候用kitty.src。



## kitty.dest

指定主文件生成目录。

格式，请参考gulp.dest。

```javascript
kitty.src(['./pages/**/*.html', '!./pages/vendor/*.html']) 
	.pipe(kitty.dest('./build/pages'))
```

*注意，其只会将主文件生成到./build/pages，cdn资源文件需要使用kitty.cdnDest。*

## kitty.cdnDest

指定cdn文件生成目录。

```javascript
kitty.src(['./pages/**/*.html', '!./pages/vendor/*.html']) 
	.pipe(kitty.dest('./build/pages'))
	.pipe(kitty.cdnDest('./build/static'))
```



# 内置模块
千万不要被下面的大量的内置模块给吓到，其实只要明白一个其他都能明白。
### loaders.beforeCompile

钩子函数，获取文件内容后立即执行该函数，非常有作用，我们常用来做一些替换操作。

```javascript
loaders.beforeCompile =  function(content){
   content = content.replace(/#apihost#/,"https://api.vkitty.org");
   return content;
};
```



### loaders.afterCompile

钩子函数，模块编译完成后执行该函数。



## loaders.tag

按照文件类型自动调用jstag、csstag、imgtag。

*注意：文件的类型不是通过后缀来判断的，[点击查看类型判断](./index.md#类型判断)*

#### loaders.jstag

用`script`标签包裹。

#### loaders.csstag

用`style`标签包裹。

#### loaders.imgtag

写入`img`标签。



## loaders.cdn

按照文件类型自动调用jscdn、csscdn、imgcdn。

```
_include('./index.less?_ac=less_cdn',{"url":"https://vkitty.org/;test.css"})
```

*注意使用后面必须有options.url。*

如何写options.url，请查看[cdn处理](./index.md#cdn处理)

#### loaders.jscdn

返回一个`script`cdn链接的引用。

#### loaders.csscdn

返回一个`link`cdn链接的引用。

### loaders.imgcdn

返回的是一个cdn地址，如果要`img`标签引用，请增加`imgtag` loader。

### loaders.mediacdn

返回的是一个cdn地址，适用于流媒体，如mp3\ogg\acc。




### loaders.mini

按照文件类型压缩，自动调用jsmini、cssmini、htmlmini。

#### loaders.jsmini

压缩js。

#### loaders.cssmini

压缩css。

#### loaders.htmlmini

压缩html。





# 进阶

##  自定义模块

vkitty自定模块非常简单，将函数挂载在kitty.loaders下面即可，函数必须返回content。

```javascript
/**
 * content 编译的内容
 * depend 处理的依赖
 * compiler 该文件的编译
*/
kitty.loaders.doSomething = function(content,depend,compiler){
  	content = content.replace(/#apihost#/,"https://api.vkitty.org");
   	return content;
};
```





## 插件

vkitty和gulp都是使用流来处理任务，所以vkitty是原生支持gulp插件的，使用方法也是一模一样，流之间用pipe连接。

下面使用了kitty.serve插件。

```javascript
const gulp = require('gulp');
const kitty = require('vkitty');
const serve = require('kitty-serve');
const loaders = kitty.loaders;

gulp.task('dev',function(){
    kitty.watch('./pages/*/*.html')
        .pipe(serve({port:8080}))
});
```



## cdn处理

通过loaders.cdn后面的options必须有url参数。

```json
{"url":"https://vkitty.org/;test.css"}//生成文件./build/static/test.css
{"url":"https://vkitty.org/;images/logo.png"}//生成文件./build/static/images/logo.png
{"url":"https://vkitty.org/"}//将生成一个以hash值为文件名的文件
```

地址使用`;`进行分割，后面的为文件的basename，将按basename生成到kitty.cdnDest('./build/static')；

我们可以在后面手工加上参数哦，比如：

```json
{"url":"https://vkitty.org/;logo.png?v=123"}//得到的地址https://vkitty.org/logo.png?v=123
```

每个都填是不是太麻烦！我们如何批量处理cdn呢？这个时候我们可以配合钩子函数来批量处理。

```javascript
index.html
_include('./index.less?_ac=less_cdn','#cdnurl#'})
index.less
body{
  background:url(_include('./index.png?_ac=cdn','#cdnurl'));
}

打包文件
loaders.beforeCompile =  function(content){
   content = content.replace(/#cdnurl#/g,"https://api.vkitty.org");
   return content;
};
```



## 类型判断

文件类型不是通过后缀来唯一判断的，他优先是看调用了什么模块，因为任何后缀的文件都可以是一个脚本文件。

文件的默认类型为`txt`。

vkitty是如何做判断的？这里以js类型为demo。

1. loaders含有jstag或者jscdn的文件。
2. 或是通过require加载进来的文件。

如果没有上述区别，最后通过后缀来判断。

js后缀：

```js后缀
['.js','.vue','.coffee']
```

也就是说上述三种后缀判断为js文件。

css后缀：

```css后缀
['.less','.sass','.css']
```

img后缀：

```
['.jpg','.jpeg','.gif','.png','.webp','.ico']
```

media后缀：

```
['.wav','.mp3','.wma','.mp4','.ogg','.ape','.acc','.asf','.rm','.ra']
```





# 使用技巧

### 解决线上压缩，开发不压缩

```javascript
gulp.task('dev',function(){
  	//开发时候不压缩
    kitty.loaders.mini = function(content){
      return content;
    }
    kitty.watch('./pages/*/*.html')
});

gulp.task('prod',function(){
    //线上直接使用就好了
	kitty.src('./pages/*/*.html')
});
```

从上述中我们很容易通过覆盖内置loader完成指定操作。

### 使用kitty-serve插件

```javascript
const gulp = require('gulp');
const kitty = require('vkitty');
const serve = require('kitty-serve');
const loaders = kitty.loaders;

gulp.task('dev',function(){
    loaders.beforeCompile =  function(content){
        content = content.replace(/#cdnurl#/,"");
        return content;
    };
    
    kitty.watch('./pages/*/*.html')
        .pipe(serve({port:8080})) //不需要kitty.dest了
});

gulp.task('prod',function(){
    loaders.beforeCompile =  function(content){
        content = content.replace(/#cdnurl#/,"../static");
        return content;
    };
    kitty.src('./pages/*/*.html')
        .pipe(kitty.dest('./build/pages'))
        .pipe(kitty.cdnDest('./build/pages/static'))
});
```

kitty-serve插件能方便启动web服务，从内存中读取内容，编译速度就更快了。

使用了kitty.dest也没有关系，照样向下执行，因为流的特性使它能够一直向下传递。



## bug反馈

https://github.com/vkitty/vkitty/issues



