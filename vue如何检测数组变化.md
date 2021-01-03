#vue如何检测数组变化？

由于 JavaScript 的限制，Vue 无法检测到以下数组变动：
1. 当你使用索引直接设置一项时，例如 vm.items[indexOfItem] = newValue
2. 当你修改数组长度时，例如 vm.items.length = newLength

原因简单来说，操作数组的方法，也就是 Array.prototype上挂载的方法并不能触发该属性的 setter，因为这个属性并没有做赋值操作。

Vue 中解决这个问题的方法，是将数组的常用方法
[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
进行重写，通过包装之后的数组方法就能够去在调用的时候被监听到

```
// 让 arrExtend 先继承 Array 本身的所有属性
const arrExtend = Object.create(Array.prototype)
const arrMethods = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
/**
 * arrExtend 作为一个拦截对象, 对其中的方法进行重写
 */
arrMethods.forEach(method => {
  const oldMethod = Array.prototype[method]
  const newMethod = function(...args) {
    oldMethod.apply(this, args)
    console.log(`${method}方法被执行了`)
  }
  arrExtend[method] = newMethod
})

export default {
  arrExtend
}
```

将 arrExtend 这个对象作为拦截器。首先让这个对象继承 Array 本身的所有属性，这样就不会影响到数组本身其他属性的使用，后面对相应的函数进行改写，
也就是在原方法调用后去通知其它相关依赖这个属性发生了变化，这点和 Object.defineProperty 中 setter所做的事情几乎完全一样，
唯一的区别是可以细化到用户到底做的是哪一种操作，以及数组的长度是否变化等等。

数组的依赖收集是给数组添加一个dep属性，存放它所依赖的watcher，当数组变化时会通知对应的watcher去更新。