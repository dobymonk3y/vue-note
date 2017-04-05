# vue 常见技巧

# 子父组件通信

每个 Vue 实例都是一个事件触发器：

* 使用 $on() 监听事件；

* 使用 $emit() 在它上面触发事件；

* 使用 $dispatch() 派发事件，事件沿着父链冒泡；

* 使用 $broadcast() 广播事件，事件向下传导给所有的后代。

```java
    <p>
        <a @remove-rule="abcRule"> 
    </p>

a : 为子组件
p ： 父组件

p 监听了 @remove-rule 事件，在 子组件中使用  this.$dispatch('remove-rule',xx); 派发事件
！注意：标签属性都不支持驼峰写法。这里派发的事件名称 与 prop 属性不一致。 必须派发非驼峰写法的事件。       
```
