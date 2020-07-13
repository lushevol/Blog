# snabbdom 笔记

[TOC]

## init

接收两个参数: `modules`(module数组)和`domApi`.  `module`是拥有一系列钩子函数的对象数据结构, 在patch的各个阶段执行. `domApi`默认是HTML的原生API, 但也可以传入自定接口(跨平台如weex实现方案). 

init 函数执行返回`patch`函数, 属于高阶函数. 

## h 函数

h 接收最多三个参数, 

参数一: selector, . (**必填**)

参数二: data, 生成VNode的数据选项. (*选填*)

参数三: children, 这层VNode的子节点. (*选填*)

- 其中, data和children的顺序不定, 且当data为string或number时, 会被认为是文本(text)数据. children类型为数组, 如果children位置不为数组的话, 会被包裹成数据赋值给children.

- 如果children中某个子元素为string/number, 则会将它转换成vnode. 

   > vnode是拥有`VNode`结构的对象

- sel 以`svg`开头的, 会增加NameSpace.

## patch

接收两个参数: oldVnode和vnode, 前者是被patch的老vnode, 后者是要patch的新vnode.

patch 函数执行过程如下:

1. pre hook
2. 若oldVnode为是Element(即sel为空), 则根据它创建一个基本的vnode(有sel和elm).
3. 如果新老vnode挂载的节点是同一个(`sameVnode: sel和key相等`), 则执行`patchVnode`方法.
4. 否则, 直接拿新vnode替换oldVnode.(将新vnode`具像化`后挂载到oldVnode父节点下, 并删除oldVnode)

## *patchVnode

核心方法, 传说中的diff算法

步骤如下:

1. 如果新vnode没有text内容

   - 如果新老vnode都有children, 且不相等: 

     执行`updateChildren`

   - 如果新vnode有children, 老的没有: 

     清空老vnode的text, 并将children一个个挂载上去

   - 如果老vnode有children, 新的没有:

     将children一个个移除

   - 都没有children, 老vnode有text:

     将text清空

2. 如果新vnode有text内容, 且新老text不一样

   - 如果老vnode有children, 则一个个移除children
   - 设置text

疑问: 会出现新vnode有text, 且有children的情况吗? 该怎么处理?

## *updateChildren

遍历diff子vnode们的方法

old: 从 oldStartIndex 0 到 oldEndIndex old.length-1

new: 从 newStartIndex 0 到 newEndIndex new.length-1

循环从start和end往中间移动, 遇到vnode为null时跳过.

1. 如果new和old到start或end vnode是sameVnode

   执行patchVnode, 并同时往中间移动.

2. 如果old的start跟new的end是sameVnode

   将old的start vnode插到old的end后面, 并同时往中间移动

3. 如果old的end跟new的start是sameVnode, 

   将old的end vnode插到old的start前面, 并同时往中间移动

4. 否则

5. 结束后, 如果new还有没遍历完的,则直接全部插入到

   如果old还有没遍历完的, 则直接全部删除.



## 参考

> VNode 和 VNodeData 的格式定义
>
```typescript
interface VNode {
  sel: string | undefined
  data: VNodeData | undefined
  children: Array<VNode | string> | undefined
  elm: Node | undefined
  text: string | undefined
  key: Key | undefined
}

interface VNodeData {
  props?: Props
  attrs?: Attrs
  class?: Classes
  style?: VNodeStyle
  dataset?: Dataset
  on?: On
  hero?: Hero
  attachData?: AttachData
  hook?: Hooks
  key?: Key
  ns?: string // for SVGs
  fn?: () => VNode // for thunks
  args?: any[] // for thunks
  [key: string]: any // for any other 3rd party module
}
```
