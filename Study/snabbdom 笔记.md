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

2. 若oldVnode为是Element(即sel为空), 则根据它创建一个基本的vnode(只保留了tag、id、class), prop和attribute等不需要.
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

***循环***从start和end往中间移动, 遇到vnode为null时跳过.

1. 如果new和old的start或end vnode是sameVnode

   执行patchVnode, 并同时往中间移动.

2. 如果old的start跟new的end是sameVnode

   执行patchNode, 将old的start vnode插到old的end后面(**?**), 并同时往中间移动

3. 如果old的end跟new的start是sameVnode, 

   执行patchNode, 将old的end vnode插到old的start前面(**?**), 并同时往中间移动

4. 如果仍没遍历完, 说明有两种可能性: 1节点是新增的; 2节点处于old的start和end中间

   创建一个old vnode的key=>index的map索引.

   然后查看new start的vnode的key是否在索引中存在:

   - 不存在, 说明是新增节点: 

     插到old startVnode的前面

   - 存在, 则找到位置发生变化的节点: 

     如果new和old的sel不一样, 则插到old startVnode前面 (同上)

     如果一样, 则patchVnode; 并将old vnode的elm插到old start前面, 并置空原来的old vnode (下次直接跳过)

   然后, new start往中间缩进

遍历结束后, 如果new还有没遍历完的,则直接全部插入到末尾

如果old还有没遍历完的, 则直接全部删除.



## modules

[通用] 对应属性都没有或者相等, 直接返回, 不处理

### attributes

create和update阶段都会updateAttribute

遍历新attrs, 将新的设置上. xml和xlink属性特殊处理; 把老attrs有但新的没有的属性都删掉.

### class

class结构是一个以string为key, boolean为value的Object.

首先把oldClass有, newClass没有的, 在classList上直接删掉; 然后把newClass里跟oldClass的flag不一样的class, 在classList上相应的add/remove.

### dataset

dataset结构是一个key/value都是string的Object

比较新老差异, 基本操作.

值得注意的是, 如果elm.dataset存在, 则直接操作它, 否则操作elm的attr, 把它当作`data-`开头的属性 (转换规则: **Camel/Pascal** => **kebab-case**). 

### eventListeners



### props

仅遍历新props.

1. 新老prop的值不相等, 且key不是“value”.

2. 新老prop的值不相等, key为“value”, 但elm的“value”值不为新的prop值.

则, 在elm上设置key的值为新的prop值.

### style



## 总结

1. 使用VNode的意义

   实际上diff算法中使用到了非常多的dom操作, 但速度依然很快, 因为在现代浏览器的优化下dom操作本来就快, 而且极短时间内的多个dom操作, 只会触发一次回流, 性能上不会消耗很大. 那为什么还要使用vnode呢? 首先是跨平台, 可以自定义renderer实现不同平台的底层操作, web上的DOM和移动平台的view; 然后就是diff算法减少了大量的重复和没必要的DOM更新, 把计算留在JS引擎中, 提升了效率.

   > 如果只是简单的dom操作, 没必要使用vnode.

2. patch函数是将新vnode跟老vnode的diff, 应用到老dom上

3. 对data中prop、attribute、event等的处理

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
