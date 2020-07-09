# snabbdom 笔记

1. init接收两个参数: `modules`(module数组)和`domApi`.  `module`是拥有一系列钩子函数的对象数据结构, 在patch的各个阶段执行. `domApi`默认是HTML的原生API, 但也可以传入自定接口(跨平台如weex实现方案). 

   init 函数执行返回patch函数, 属于高阶函数. 

2. h 函数

   h 接收最多三个参数, 

   参数一: selector, 必填. 