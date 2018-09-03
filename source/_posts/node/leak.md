---
title: 内存泄漏
date: 2018-05-06
tags: [node]
categories: node
---

### 一、几种典型的内存泄漏

#### 1.1、全局变量的无限制使用
```js
var LeakArr = [];
function leakObj() {}; // 定位此变量
// 1、数组无限增长
app.get('/LeakArr', function(req, res) {
  for (var i = 0; i < 1000; i++) { LeakArr.push(new leakObj())}
  res.send('LeakArr');
})
```

模拟接口的调用
```sh
ab -c 10 -n 10000 http://127.0.0.1:3000/LeakArr
```

内存快照
![image](/images/leak/arr.png)

#### 1.2、事件的监听事件, 内存使用过大
```js
eventEmitter.setMaxListeners(0) // https://nodejs.org/docs/latest/api/events.html#events_emitter_setmaxlisteners_n
app.get('/eventEmitter', function(req, res) {
  eventEmitter.on('request', function (){})
  res.send('event')
})
```
模拟接口的调用
```sh
ab -c 10 -n 10000 http://127.0.0.1:3000/eventEmitter
```

#### 1.3、闭包的使用
```js
var leakThing = null;
var creatThing = function() {
  var leak = leakThing
  var unsetd = function () {
    if(leak) {}
  }
  return leakThing = {
    childs: new Array(1000*1000),
    getChilds: function() {
      return this.childs.length
    }
  }
}
app.get('/closure', function(req, res) {
  var thing = creatThing()
  res.send('closure');
});
```
模拟接口的调用
```sh
ab -c 10 -n 1000 http://127.0.0.1:3000/closure
```

内存快照

![image](/images/leak/closure.png)

### 二、定位的方法

#### 2.1、node inspector
  V8现在已经可以直接 [inspector](https://nodejs.org/api/debugger.html#debugger_v8_inspector_integration_for_node_js) 了, 只需要执行 `node --inspect ***.js` 就可以配合chrome查看内存信息

具体步骤
1. `node --inspect ***.js`
2. 打开`chrome://inspect/#devices`
3. 打开开发者工具 `profile`
4. 点击 `Take SnapShot`

这样就可以根据右侧的信息，查看哪个对象暂用的内存过大

#### 2.2、easy-Monitor定位
 [npm介绍](https://www.npmjs.com/package/easy-monitor)
 [github](https://github.com/hyj1991/easy-monitor)
#### 2.3、heapdump/v8-profile


### 参考文件
1. [如何定位 Node.js 的内存泄漏](http://taobaofed.org/blog/2016/04/16/how-to-find-memory-leak/)
2. [使用Chrome+node-inspector查找NodeJS内存泄漏](http://www.cnblogs.com/ldlchina/p/4762036.html)
3. [轻松排查线上Node内存泄漏问题](https://cnodejs.org/topic/58eb5d378cda07442731569f) 
4. [node-debug-tutorial](http://i5ting.github.io/node-debug-tutorial/)
5. [V8 内存浅析](https://zhuanlan.zhihu.com/p/33816534)
