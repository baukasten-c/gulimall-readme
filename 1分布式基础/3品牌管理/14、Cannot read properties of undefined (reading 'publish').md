# Cannot read properties of undefined (reading 'publish')

1、品牌管理的关联分类模块报错 Cannot read properties of undefined (reading 'publish')

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692853094243-0d64a780-f2af-4514-adf1-f5aca2e38e50.png)

2、具体流程：

2-1、父组件通过下方代码来使用子组件的级联选择器。用户在级联选择器中选择了一个分类路径，导致 catelogPath 的值发生了变化。

```vue
 <category-cascader :catelogPath.sync="catelogPath"></category-cascader> 
```

2-2、这个变化触发了 catelogPath 的 Watcher 函数，在这个函数中，将 catelogPath 的值同步到 paths 上，以备后续处理。

```vue
catelogPath(v) {
  this.paths = this.catelogPath;
}
```

2-3、因为 paths 属性的值发生了变化，接着会触发 paths 的 Watcher 函数，将值传递回父组件

```vue
paths(v) {
  this.$emit("update:catelogPath", v);
  this.PubSub.publish("catPath", v);
}
```

3、父组件中并没有 @update:catelogPath，为什么可以接收到子组件传递的值呢？

因为父组件中使用了 .sync 修饰符，它实际上是将以下两个操作合并在一起：

3-1、将父组件的属性传递给子组件： 通过 :propName 的形式将父组件的属性传递给子组件作为 prop。例如：:catelogPath。

```vue
props: {
    catelogPath: {
      type: Array,
      default() {
        return [];
      },
    },
},
```

3-2、在子组件内部修改属性并触发事件： 子组件内部通过修改一个与 prop 同名但带有 update: 前缀的事件，来触发父组件中对应的属性更新。例如，在子组件内部使用 this.$emit('update:catelogPath', newValue) 来触发更新。

4、因为控制台报错 Cannot read properties of undefined (reading 'publish')，我们首先把它注释掉，发现级联选择器选中分类路径后不会报错了，新增功能可正常使用，即 this.$emit("update:catelogPath", v); 与 this.PubSub.publish("catPath", v); 功能相同，使用其中一个即可

5、改为使用 this.PubSub.publish("catPath", v);

5-1、输入 npm install --save pubsub-js 安装依赖

5-2、在 brand.vue 和 category-cascader.vue 里都导入 import PubSub from 'pubsub-js'

5-3、在 brand.vue 中将 this.PubSub.publish("catPath", v); 改为 PubSub.publish("catPath", v); 来发布消息

5-4、在category-cascader.vue 中添加代码来订阅消息

```vue
mounted(){
  PubSub.subscribe("catPath", (msg, data) => {
    this.catelogPath = data
  })
},
```

5-5、测试，发现可以正常使用

6、个人更倾向于使用 this.$emit("update:catelogPath", v); 因为相关代码少，而且不需要取消订阅