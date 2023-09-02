---
layout: post
title:  "[翻译]如何在Flutter中使用async/await？"
date:   2023-04-21 02:05:45 +0800
categories: Flutter
---

> 导读：这是一篇英文博客的译文，原文通俗易懂，内容基础，我看完后获益匪浅，因此想到把它翻译成中文，供大家学习。也顺便练习了自己的英文翻译水平😛
原文链接：<https://sarunw.com/posts/how-to-use-async-await-in-flutter/>  以下是正文内容：

最近工作中有机会使用到Dart来开发Flutter应用

Dart里最让我疑惑的事情就是异步编程了。在swift中，我们只用到async和await，但是在Dart中（可能还有其他语言）我们还有Future（或者Promise）对象

对我来说，Future、async、await的组合十分容易出错，也容易让人迷惑

本文将向你展示一系列异步实现，会着重强调一些坑点，最后再给出这三个组件（Future、async、await）的用法总结

## 一、什么是Future

Future是一个对象，表示异步操作的结果，可以有两种状态，未完成和已完成

本文不会详细介绍未完成状态的案例

Future是我们在Dart中表示异步操作的一个标识，异步函数就是一个返回Future的函数。

``` dart
Future<String> getWeatherForecast() {
  return Future.delayed(Duration(seconds: 2), () => "Partly cloudy");
}
```

注意，在这里我们不需要使用`async`来表示异步函数

## 二、async和await

`async`和`await`是一组关键字，它们使异步操作看起来像是同步操作。为了理解这一点，先让我们看看如何通过回调处理异步操作的结果

我们一般使用`then`方法来处理Future的值，它接受一个回调参数，在Future完成时调用

在下面的示例中，我们获取了天气预报的结果并在回调中打印这个值

``` dart
Future<String> getWeatherForecast() {
  return Future.delayed(Duration(seconds: 2), () => "Partly cloudy");
}

void fetchWeatherForecast() {
  print("start: fetchWeatherForecast");
  final forecast = getWeatherForecast();
  forecast.then(
    (value) => print("fetchWeatherForecast: $value"),
  );
  print("end: fetchWeatherForecast");
}

void main(List<String> arguments) {
  print('start: main');
  fetchWeatherForecast();
  print('end: main');
}
```

打印结果如下

``` dart
start: main
start: fetchWeatherForecast
end: fetchWeatherForecast
end: main
// Wait for 2 seconds
fetchWeatherForecast: Partly cloudy
```

可以看到代码很难预测，每个print语句都没按照定义的顺序打印。让我们看看async和await如何帮我们解决这个问题

## 三、什么是async

async只有两个功能

- 将任何函数转换为async函数
- 自动将返回语句包裹上Future

可以通过在函数体前添加`async`关键字来声明一个async函数

``` dart
Future<int> futureInt() async {
  // 这里不需要显示返回 Future.value(1)，因为async关键字会自动进行包装
  return 1;
}
```

如果尝试删除async关键字，会得到下面的报错

> A value of type ‘int’ can’t be returned from the function ‘futureInt’ because it has a return type of ‘Future’  

请注意：async函数不完全等同于异步函数，你用async声明一个同步函数也不会报错

``` dart
void noFuture() async {
  // ...
}
```

大多数情况下，async关键字只是通过强制返回为Future来帮你把函数转化为异步

如果返回类型不是Future，就会得到以下错误
> Functions marked ‘async’ must have a return type assignable to ‘Future’.  
> Try fixing the return type of the function, or removing the modifier ‘async’ from the function body.  

``` dart
int futureInt() async {
  return 1;
}
```

但是，如果返回类型为void，就不会报这个错

``` dart
void noFuture() async {
  // 这段代码不会报错
}
```

上面这段代码即使使用了async关键字也会被看做一个同步函数，我不知道这是一个bug还是背后有什么设计哲学。但至少应该意识到这一点
**async关键字并不代表着异步函数**

## 四、什么是await

你可以把await看作是then的一个语法糖，它让异步操作看起来像是同步的
await会等待Future完成后再执行后面的语句，这就让异步操作看起来像是同步的

让我们使用`async`和`await`来修改`fetchWeatherForecast`方法
这是我们先前的实现：

``` dart
void fetchWeatherForecast() {
  print("start: fetchWeatherForecast");
  final forecast = getWeatherForecast();  
  forecast.then(
    (value) => print("fetchWeatherForecast: $value"),
  );
  print("end: fetchWeatherForecast");
}
void main(List<String> arguments) {
  print('start: main');
  fetchWeatherForecast();
  print('end: main');
```

下面是使用`async/await`的实现：

``` dart
// 1 我们添加async关键字使方法支持await
Future<void> fetchWeatherForecast() async {
  print("start: fetchWeatherForecast");
  // 2 我们使用await等待函数返回的结果
  final forecast = await getWeatherForecast();
  // 3 上面函数结果返回后，这行代码才会被执行
  print("fetchWeatherForecast: $forecast");
  print("end: fetchWeatherForecast");
}
void main(List<String> arguments) {
  print('start: main');
  fetchWeatherForecast();
  print('end: main');
```

两组代码的打印结果如下：

``` dart
// Use then
start: main
start: fetchWeatherForecast
end: fetchWeatherForecast
end: main
// Wait for 2 seconds
fetchWeatherForecast: Partly cloudy

// Use async/await
start: main
start: fetchWeatherForecast
end: main
// 2 seconds
fetchWeatherForecast: Partly cloudy
end: fetchWeatherForecast
```

如你所见，`fetchWeatherForecast: Partly cloudy`和`end: fetchWeatherForecast`都在await执行完后才打印
这两个方法的输出是不同的，then方法在启动后会立即打印`end: fetchWeatherForecast`
其实async和await只是语法糖，我们稍微修改一下也能得到同样的打印

``` dart
Future<void> fetchWeatherForecast() async {
  final forecast = getWeatherForecast();
  print("start: fetchWeatherForecast");
  forecast.then(
    (value) {
      print("fetchWeatherForecast: $value");
      // 1 我们把打印移到成功的回调中
      print("end: fetchWeatherForecast");
    },
  );
}
```

你可以把await后的语句看做是then回调中的语句

我故意没有在main函数的`fetchWeatherForecast()`方法前加await，就是为了展示async/await是和then等效的

你也可以给main函数添加async，`fetchWeatherForecast()`前添加await，这样代码就会完全按照顺序执行了

``` dart
Future<String> getWeatherForecast() {
  return Future.delayed(Duration(seconds: 2), () => "Partly cloudy");
}
Future<void> fetchWeatherForecast() async {
  print("start: fetchWeatherForecast");
  final forecast = await getWeatherForecast();
  print("fetchWeatherForecast: $forecast");
  print("end: fetchWeatherForecast");
}
void main(List<String> arguments) async {
  print('start: main');
  await fetchWeatherForecast();
  print('end: main');
}
```

打印如下：

``` dart
start: main
start: fetchWeatherForecast
// 2 seconds
fetchWeatherForecast: Partly cloudy
end: fetchWeatherForecast
end: main
```

注意：
1、异步函数的特征是返回Future，而不是说有async关键字
2、你可以尽情地像使用同步函数一样使用异步函数，因此使用时必须小心

## 五、Dart异步编程综述

1、异步函数是一个返回Future类型的函数
2、把await放在异步函数前，等待Future结果返回后才会执行后续的代码
3、函数体前加async，说明函数支持await
4、async函数会自动将返回值包装为Future（如果还没包装的话）

关于Dart中的异步编程，我想这就是你需要了解的全部内容
