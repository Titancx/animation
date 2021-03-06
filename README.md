# 测试case
查看[百度脑图](http://naotu.baidu.com/file/c256288088a1359a6dcdadd90cc6b0cc?token=086cb65c016d2bd2)附件

# 测试页面
见demo

# 设计目的
  - 功能
    - 提高动画性能，css3 transition实现；
    - 支持动画播放/暂停；
  - 兼容
    - 不支持transition，通过setTimeout实现；
  - 写法
      - 支持链式写法；
      - 支持非链式写法；
  - 执行顺序
    - 支持串行动画；
    - 支持并行动画；
    - 支持串行&并行结合动画（JQuery&Zepto不支持）；
    - 支持多dom动画（通过类选择器，默认为并行）；
  - 类型
    - 支持普通动画；
    - 支持transform动画；
    - 不支持keyframe动画；
  - 回调
    - 支持串行回调；
    - 支持并行回调；
    - 支持并行动画全部执行完成的回调；

# 案例
- 单对象串行；
```
    animate(propDomCallbackBtn1, {
        width: "70%"
    }, 1000, "ease", function () {
        console.log("串行1");
    })
    .animate(propDomCallbackBtn2, {
        height: 200
    }, 3000, "ease", function () {
        console.log("串行2");
    });
```
  
- 单对象并行；
```
    animate(propDomCallbackBtn1, {
        width: "70%"
    }, 1000, "ease", function () {
        console.log("并行1");
    }, 2000, 1)
    .animate(propDomCallbackBtn2, {
        height: 200
    }, 3000, "ease", function () {
        console.log("并行2");
    }, 0, 1)
    .endAnimaion(function() {
        console.log("并行动画全部执行完成");
    });
```
- 单对象串行&并行；
```
    animate(propDomCallbackBtn1, {
        width: "70%"
    }, 1000, "ease", function () {
        console.log("并行1");
    }, 2000, 1)
    .animate(propDomCallbackBtn2, {
        height: 200
    }, 3000, "ease", function () {
        console.log("并行2");
    }, 0, 1)
    .endAnimaion(function() {
        console.log("并行动画全部执行完成");
    })
    .animate(propDomCallbackBtn2, {
        height: 200
    }, 3000, "ease", function () {
        console.log("串行1");
    });
```    
- 多对象并行；
```
    animate(propDomCallbackBtn1, {
        width: "70%"
    }, 1000, "ease", function () {
        console.log("并行1");
    }, 2000, 1)
    .animate(propDomCallbackBtn1, {
        height: 200
    }, 3000, "ease", function () {
        console.log("并行2");
    }, 0, 1)
    .endAnimaion(function() {
        console.log("并行动画全部执行完成1");
    });

    animate(propDomCallbackBtn2, {
        width: "70%"
    }, 1000, "ease", function () {
        console.log("并行3");
    }, 2000, 1)
    .animate(propDomCallbackBtn2, {
        height: 200
    }, 3000, "ease", function () {
        console.log("并行4");
    }, 0, 1)
    .endAnimaion(function() {
        console.log("并行动画全部执行完成2");
    });
```
# 参数
- dom: dom元素，可以是一个元素，也可以是一个dom数组，不能为空；
- property: 属性对象，不能为空；
- duration: 动画持续时间，如果不设置默认为0.4s；
- easing: 动画执行的形式, 如"ease, ease-in, ease-out, ease-in-out, linear, cubic-bezier", 默认为"linear";
- callback: 回调函数，在动画执行完成之后执行，分为两种，一种为单个动画执行完成之后的回调，另一种是所有并行动画执行完成的回调；
- delay: 延迟时间，如果不设置默认为0s；
- isAsync: 是否动画是并行，0为串行，1为并行;

# 注意点
- 并行动画需要调用endAnimation才能结束并执行；
- dom参数为数组默认为并行动画，需要调用endAnimation；
- 非链式写法是创建了多个对象，所以表现为并行动画，不能通过isAsync控制为串行；

# 设计方案概述
  - 存储方案：单个对象串并行动画通过一个队列来进行存储；
  - 多对象动画方案：通过生成多个对象来各自管理；
  - 单对象动画方案：每个对象的串并行动画通过队列存储，以"#"分隔，每两个"#"之间代表一组并行动画，如果只有一个，则表现为串行形式；

# 详细设计方案
- 关键类
  - 入口类：每次单独（串行方式）调用一个`animate`函数时都会经过该类，他的主要作用是生成一个新的动画对象，并返回动画管理类，从而支持并行工作和链式调用，下面是其实现：
  
      ![入口类](http://upload-images.jianshu.io/upload_images/2483150-78c56ed41879b1cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
      
      ![入口类实现](http://upload-images.jianshu.io/upload_images/2483150-a3342c7906aa7f16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  - 动画管理类：该类是管理每个对象生成的动画，包含三个主要属性和方法；
      - `isRuning`：当前动画执行状态，执行中/执行完成；
      - `asyncQueue`：动画队列，存储串并行动画对象；
      - `endCallback`：并行动画全部执行完成的回调队列；
      - `animate(ele, property, duration, easing, callback, delay, isAsync)`：动画开始入口；
      - `endAnimation(fn)`：结束并行动画，其中可以传入回调；
      - `stop()`：停止当前动画；

      ![动画管理类](http://upload-images.jianshu.io/upload_images/2483150-0e3ea78dc31dc08c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 动画方案时序图：粗略的分为这几个块，starUML不太会用，有的生命周期可能画的不太对，先将就着看看
![动画时序图](http://upload-images.jianshu.io/upload_images/2483150-1366cb25b7e5bc23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  - 第一步：首先一个动画对象进入之后会先初始化一个动画对象；
  - 第二步：初始化后进入动画管理中心，进行单个动画管理；
  - 第三步：动画正式开始之前会对动画的参数进行筛选和处理，比如集合属性的单位（`'px'`）处理，浏览器vendor处理（`'webkit', 'moz', 'ms', 'o', 'Webkit', 'Moz', 'O'`），是否支持transition处理等；
  - 第四步：创建动画对象，并将其加入动画队列中；
  - 第五步：开始执行动画；
  - 第六步：动画执行完成之后进行该动画回调；
  - 第七步：所有回调都执行完成之后进行并行动画终止回调的执行；
  - 第八步：如果队列中仍然存在动画对象，执行下一个，跳转至步骤一；

# 设计棘手点
- transitionend多次执行：transitionend执行时机是每一个属性执行完成就会执行，所以你的动画有几个属性，他就会执行几次。这个比较好解决，在执行时remove了transitionend事件监听即可；但是如果多个并行动画执行在一个dom上时就蛋疼了，单纯remove监听是做不到的，可以通过判断动画delay+duration来判断是否是当前动画；
- 不支持transition怎么办：该实现方案采用的方式是，transition和setTimeout会同时执行，在一次回调处理时会clearTimeout和remove监听，从而做到双保险的作用；

# License
MIT License
