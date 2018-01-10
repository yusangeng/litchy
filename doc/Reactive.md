# Reactive - 可监听属性支持

提供了一套可监听属性实现, 类似于VUE属性或mobx. 

## 引用方式

### mixin方式

*注意*: Reactive会自动引入Eventable, 不要重复引入.

``` js
import Reactive from 'litchy/lib/mixin/Reactive'
import mix from 'litchy/lib/mix'

class Base {
  // ...
}

class Foobar extends mix(Base).with(Reactive) {
  constructor() {
    super()
    this.initReactive({apple: 0})
  }

  someMethod() {
    this.apple = 1 // 此处会发出prop-change事件以及prop-change:apple事件
  }
}
```

### 装饰方式

``` js
import reactive from 'litchy/lib/decorator/reactive'
import props from 'litchy/lib/decorator/props'

@reactive
class Foobar {
  @props({
    apple: 0
  })

  someMethod() {
    this.apple = 1 // 此处会发出prop-change事件以及prop-change:apple事件
  }
}
```

## 事件

Reactive依赖Eventable提供的事件收发机制, 当预先定义的可监听属性发生变化时, 会发两个事件: 
* prop-change
* prop-change${name}, 其中name为属性名称

## 属性

可监听属性均可通过标准的属性方式读写(计算属性不可写). 另外, Reactive提供了以下属性: 

### batch

当处于批量模式时为true, 否则为false. 

## 方法

### dispose()

清理资源.

### initReactive(props, computed = {}, reactions = [])

初始化可监听属性以及属性变化时的反馈方法. 建议在子类构造函数中调用.

### getPropValue(name)

获取可监听属性值.

### setPropValue(name, value)

设置可监听属性值.

### getPropValues()

批量获取可监听属性值.

### setPropValues(map)

批量设置可监听属性值.

### startBatch()

开启批量模式.

### endBatch()

结束批量模式.

## 装饰

### @reactive

在类定义前添加此装饰, 你定义的类会自动mixin Reactive.

### @props(propDefs)

在类定义中添加此装饰, 会根据propDefs参数添加可监听属性. 

#### 参数

propDefs {Object} 每一个字段表示一个可监听属性, key为属性名, value为属性初始化或者初始化函数, 如果是一个函数, 则调用函数得到初始化值, 如果是其他值, 则clone得到初始化值. 

#### 例子

``` js
@props({
  apple: 0,
  banana: [1, 2, 3],
  orange: _ => Math.random()
})
```

### @computed

在属性getter定义前添加此装饰, 会将此属性转换为计算属性. 所谓计算属性, 就是依赖于其他可监听属性计算出的属性. 当其他可监听属性改变时, 计算属性会跟着改变. 同时计算属性也可以被其他计算属性监听. 

@computed装饰会通过运行getter自动分计算属性的析依赖关系. 

*注意*: 计算属性不能定义setter. 

#### 例子

``` js
@computed
get apple() {
  // banana和orange为可监听属性
  return this.banana _ this.orange
}
```

### @action

有时我们希望批量修改属性值, 在成员方法定义前添加此装饰, 会在方法入口处添加`startBatch`调用, 通知Reactive进入批量状态. 一个被装饰为action的方法, 修改多个属性值时, 每个被修改的属性只会发送一次`prop-change`事件, 每个监听它们的计算属性也只会发送一次`prop-change`事件, 监听它们的reaction也只会被调用一次. 

#### 例子

``` js
@reactive
class Foobar {
  @props({
    apple: 0,
    banana: [1, 2, 3],
    orange: _ => Math.random()
  })

  @computed
  get cherry () {
    return this.apple + this.banana.reduce((prev, el) => {
      return prev + el
    }, 0) + this.orange
  }

  @action
  someMethod() {
    // 修改这三个属性, cherry属性只会发出一次prop-change事件
    this.apple = 1
    this.banana = [2, 3, 4]
    this.orange = Math.random()
  }
}
```

### @reaction

有时我们希望监听某些属性的变化, 并做出某些反馈. 一个显而易见的方法是监听`prop-change`事件并在回调函数中执行反馈. 而@reaction是一个语法糖, 实现了自动注册事件回调以及自动分析依赖关系. 

#### 例子

``` js
@reactive
class Foobar {
  @props({
    apple: 0
  })

  someMethod() {
    this.apple = 1
  }

  // apple属性改变后, 这个方法会自动运行
  @reaction
  onAppleChange() {
    console.log(this.apple)
  }
}
```


