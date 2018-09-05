---
title: gulp 构建总结
date: 2018-01-21 21:43:45
tags:
	- 总结
---

> 项目背景

本文记录下自己使用gulp构建的配置。

> 代码配置

```
var gulp = require( 'gulp' );
var htmlmin = require( "gulp-htmlmin" );
var uglify = require( 'gulp-uglify' );
var livereload = require( 'gulp-livereload' );
var babel = require( 'gulp-babel' );
var browserify = require( 'browserify' );
var source = require( 'vinyl-source-stream' );
var postcss = require( 'gulp-postcss' );
var autoprefixer = require( 'autoprefixer' );
var cssnano = require( 'cssnano' );
var less = require( 'gulp-less' );

//  压缩HTML
gulp.task( 'minify-html', function () {

  var options = {
    removeComments: true,  //清除HTML注释
    collapseWhitespace: true,  //压缩HTML
    collapseBooleanAttributes: true,  //省略布尔属性的值 <input checked="true"/> ==> <input checked />
    removeEmptyAttributes: true,  //删除所有空格作属性值 <input id="" /> ==> <input />
    removeScriptTypeAttributes: true,  //删除<script>的type="text/javascript"
    removeStyleLinkTypeAttributes: true,  //删除<style>和<link>的type="text/css"
    minifyJS: true,  //压缩页面JS
    minifyCSS: true  //压缩页面CSS
  };
  gulp.src( '*.html' )
    .pipe( htmlmin( options ) )
    .pipe( gulp.dest( 'dist' ) )
    .pipe( livereload() );

} );

//  把图片放到dist中
gulp.task( 'img', function () {
  return gulp.src( 'img/*' )
    .pipe( gulp.dest( "dist/img" ) );
} )

//  给css使用自动加前缀插件，并压缩css，插件来源于postcss
gulp.task( 'css', function () {
  var plugins = [
    autoprefixer( { browsers: [ 'last 2 version', '> 0.1%' ] } ),
    cssnano()
  ];

  return gulp.src( 'style/*' )
    .pipe( less() )
    .pipe( postcss( plugins ) )
    .pipe( gulp.dest( 'dist/style' ) );
} );

//  给js使用babel,并压缩
gulp.task( 'script', function () {
  gulp.src( 'js/*.js' )
    .pipe( babel( {
      presets: [ 'env' ]
    } ) )
    .pipe( uglify() )
    .pipe( gulp.dest( 'dist/js' ) )
    .pipe( livereload() );
} )

// browserify预编译，打包js
gulp.task( "browserify", function () {
  var b = browserify( {
    entries: "./dist/js/main.js"
  } );

  return b.bundle()
    .pipe( source( "bundle.js" ) )
    .pipe( gulp.dest( "dist/js" ) );
} );

//  监听代码热更新并重新执行任务
gulp.task( 'watch', function () {
  livereload.listen({port: 12345});
  gulp.watch( 'style/*', [ 'css' ] );
  gulp.watch( 'wxphp/*', [ 'wxphp' ] );
  gulp.watch( 'wxshare/*', [ 'wxshare' ] );
  gulp.watch( 'js/*', [ 'script', 'browserify' ] );
  gulp.watch( 'index.html', [ 'minify-html' ] );
  gulp.watch( 'cj.html', [ 'cj' ] );
} )

gulp.task( 'default', [ 'css', 'script', 'minify-html', 'img', 'browserify', 'watch' ] );

```
（完）