## 1. 什么是gulp
    gulp是可以自动化执行任务的工具 在平时开发的流程里面,一定有一些任务需要手工重复得执行,只要你觉得有些动作是要重复去做的,就可以把这些动作创建成一个gulp任务,然后在特定的条件下自动执行. 比如在less源文件发生改变后自动编译成css文件
    
    
## 4个API 
```
var gulp = require('gulp')
//task用来定义一个任务
gulp.task('hello', function () {
    console.log('hello')
})
//命令行运行： gulp hello的任务

//gulp的工作方式
gulp.task('html', function () {
    //通过src获取源文件  *.html  所有html文件
  gulp.src('./app/index.html')  //获取可读流 -> 目标都可写流   pipe里面放的是流，流和以前流的单位不一样，这个是以文件为单位，文件流，包括：{filename:'文件名',content:内容}
      .pipe(gulp.dest('./build'))  //dest目标
})
//监控
gulp.task('watch', function () {
    //监控原文件变化，当他发生变化之后，执行对应的任务数组
    gulp.watch('./app/*.html', ['html'])
})
gulp.task('watch2', function () {
    //监控原文件变化，当他发生变化之后，执行对应的任务数组
    gulp.watch('./app/*.html', ['html', function (event) {
        console.log(event) //event:  type:变化类型   path:改变的文件的路径
    }])
})
```

## gulp-less
```python
var less = require('gulp-less')
//负责把less编译成css
gulp.task('css', function () {
    //先得到输入文件，然后导入到插件中，然后在保存到目录下保存
    gulp.src('./app/less/*.less')
        .pipe(less()) //先调用，为了传参
        .pipe(gulp.dest('./build/css')) //目标目录
})
```

## 自定义加载各个模块去掉gulp并以驼峰命名
```
var pack = require('./package.json')
//console.log(pack.devDependencies)
function load() {
    let $ = {}
    for (var name in pack.devDependencies) {
        if(name.startsWith('gulp-')){
            $[name.slice(5).replace(/-(\w)/g, function () {
                return arguments[1].toUpperCase()
            })] = require(name)
        }
    }
    return $
}
module.exports = load
```


##
## gulp自动化打包项目
```
var gulp = require('gulp')
var $ = require('gulp-load-plugins')()
//css
gulp.task('css', function () {
    gulp.src('./src/less/*.less')
        .pipe($.less())
        .pipe($.concat('all.css'))
        .pipe(gulp.dest('./build/css'))
        .pipe($.cleanCss())     //压缩css
        .pipe($.rename(function (file) { //'all.min.css' arguments: dirname:文件目录  basename:   extname:扩展名
            //all.css   all.min.css
            file.basename += '.min'   //重命名
        }))
        .pipe(gulp.dest('./build/css'))  //保存重命名的文件
})
//js
gulp.task('js', function () {
    gulp.src('./src/js/*.js')
        .pipe($.babel({presets:['es2015']})) //ES6 -> ES5 装banbel-preset-2015
        .pipe($.concat('all.js'))   //合并一个js文件
        .pipe(gulp.dest('./build/js'))  //先保存一份可读的js文件
        .pipe($.uglify())   //对文件进行压缩
        .pipe($.rename(function (file) {
            file.basename += '.min'   //重命名
        }))   //保存一份重命名的文件
        .pipe(gulp.dest('./build/js')) //保存压缩后的js文件
})
//img
gulp.task('img', function () {
    gulp.src('./src/imgs/*')
        .pipe(gulp.dest('./build/imgs'))
})
//html
gulp.task('html',['css', 'img', 'js'], function () {  //html依赖css js img -> 依赖任务
    var source = gulp.src('./src/index.html')  //获取源文件流
    var resource = gulp.src(['./build/css/all.css', './build/js/all.js'])  //获取资源文件文件流
    source.pipe($.inject(resource, {addRootSlash: false, ignorePath: 'build'})) //向源文件中插入文件流 css js | addRootSlash不要在路径前加/    |  ignorePath忽略前导路径（忽略前面的build）
          .pipe(gulp.dest('./build'))
          .pipe($.connect.reload())  //通知浏览器执行热刷新
})
//serve
gulp.task('serve',  function () {
    $.connect.server({ //启动一个express http服务器
        port: 8080,  //设置端口为8080
        root: './build',  //设置静态文件根目录
        livereload: true  //启动浏览器自动刷新功能
    })
})
gulp.task('watch', function () {
    gulp.watch('./src/*', ['html', 'css', 'js']) //监控原文件变化，当他发生变化之后，执行对应的任务数组
})
gulp.task('default', ['html', 'serve', 'watch'])  //组合任务  会默认执行default
```
