# [vue-better-scroll](https://github.com/huangjincq/vue-better-scroll)

> A vue plugins based on [better-scroll](https://github.com/ustbhuangyi/better-scroll)

> 下拉刷新、上拉加载的场景很常见， [Vux](https://github.com/airyland/vux) 的 Scroller 组件作者不推荐使用也停止维护了，最后决定根据better-scroll封装成自己的vue组件，作者也提供了详细的教程。


# 滚动原理

由于 better-scroll 的滚动原理为：在滚动方向上，第一个子元素的长度超过了容器的长度。

那么对于 Scroll 组件，其实就是内容元素 .list-content 在滚动方向上的长度必须大于容器元素 .wrapper。

任何时候如果出现无法滚动的情况，都应该首先查看内容元素 .list-content 的元素高度/宽度是否大于容器元素 .wrapper 的高度/宽度。这是内容能够滚动的前提条件。如果内容存在图片的情况，可能会出现 DOM 元素渲染时图片还未下载，因此内容元素的高度小于预期，出现滚动不正常的情况。此时你应该在图片加载完成后，比如 onload 事件回调中，手动调用 vue-better-scroll 组件的 refresh() 方法，它会重新计算滚动距离。

# Use Setup

### Install better-scroll
```javascript
yarn add better-scroll
// or
npm install better-scroll -s
```

### Vue mount

```javascript
// import
import Vue from 'vue'
import VueBetterScroll from 'better-scroll'

// or require
var Vue = require('vue')
var VueBetterScroll = require('better-scroll')

// mount with global
Vue.use(VueBetterScroll)

// mount with component(can't work in Nuxt.js/SSR)
import { VueBetterScroll } from 'better-scroll'

export default {
  components: {
    VueBetterScroll
  }
}


// 或者直接导入js文件
<script src="./dist/vue-better-scroll.js"></script>

```

### Use in SPA
```html
<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch } from "vue-property-decorator";
import BScroll from "better-scroll";

@Component({})
export default class Scroll extends Vue {
  @Prop({
    default: () => {
      return 1;
    }
  })
  probeType!: number;
  @Prop({
    default: () => {
      return true;
    }
  })
  click!: boolean;
  @Prop({
    default: () => {
      return false;
    }
  })
  listenScroll!: boolean;
  @Prop({
    default: () => {
      return null;
    }
  })
  data!: Array<any>;
  @Prop({
    default: () => {
      return false;
    }
  })
  pullup!: boolean;
  @Prop({
    default: () => {
      return false;
    }
  })
  beforeScroll!: boolean;
  @Prop({
    default: () => {
      return 20;
    }
  })
  refreshDelay!: number;
  scroll: any;


  mounted() {
    console.log("ok");
    setTimeout(() => {
      this._initScroll();
    }, 20);
  }
  _initScroll() {
    if (!this.$refs.wrapper) {
      return;
    }
    this.scroll = new BScroll(this.$refs.wrapper, {
      probeType: this.probeType,
      click: false,
      scrollbar: true,
      mouseWheel: {
        speed: 30,
        invert: false,
        easeTime: 40
      },
      pullUpLoad: {
        threshold: 10 //当上拉距离超过盒子高度的10px的时候,就派发一个上拉加载的事件
      }
    });
    console.log(this.scroll);
    if (this.listenScroll) {
      let me = this;
      this.scroll.on("scroll", (pos: any) => {
        me.$emit("scroll", pos);
      });
    }

    if (this.pullup) {
      this.scroll.on("scrollEnd", () => {
        if (this.scroll.y <= this.scroll.maxScrollY + 50) {
          this.$emit("scrollToEnd");
        }
      });
    }

    if (this.beforeScroll) {
      this.scroll.on("beforeScrollStart", () => {
        this.$emit("beforeScroll");
      });
    }
  }
  disable() {
    this.scroll && this.scroll.disable();
  }
  enable() {
    this.scroll && this.scroll.enable();
  }
  refresh() {
    this.scroll && this.scroll.refresh();
  }
  scrollTo() {
    this.scroll && this.scroll.scrollTo.apply(this.scroll, arguments);
  }
  scrollToElement() {
    this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments);
  }
}
</script>


<style lang="scss" scoped>
</style>

```

### Attributes:

参数 | 说明 | 类型 | 可选值 | 默认值
|---|---|---|---|---|
probeType | 派发scroll事件的条件 | Number | 1、2、3 | 1 |
click | better-scroll 会派发一个 click 事件 | Boolean |  | true
listenScroll | 是否监听滚动，开启后才能派发scroll事件| Boolean |  | false
listenBeforeScroll | 是否监听滚动之前，开启后才能派发beforeScrollStart事件| Boolean |  | false
direction | 滚动方向| String | horizontal、vertical | vertical
scrollbar | 这个配置可以开启滚动条。当设置为 true 或者是一个 Object 的时候，都会开启滚动条，默认是会 fade 的 | Boolean or Object | {fade: true},  | false
pullDownRefresh | 这个配置用于做下拉刷新功能。当设置为 true 或者是一个 Object 的时候，可以开启下拉刷新，可以配置顶部下拉的距离（threshold） 来决定刷新时机以及回弹停留的距离（stop）| Boolean or Object | {threshold: 90,stop: 40}, | false
pullUpLoad | 这个配置用于做上拉加载功能。当设置为 true 或者是一个 Object 的时候，可以开启上拉加载，可以配置离底部距离阈值（threshold）来决定开始加载的时机| Boolean or Object | { threshold: 0, txt: { more: '加载更多',noMore:'没有更多数据了'} } | false
startY | 纵轴方向初始化位置 | Number |  | 0
freeScroll | 有些场景我们需要支持横向和纵向同时滚动，而不仅限制在某个方向，这个时候我们只要设置 freeScroll 为 true 即可 | Boolean |  | false

### Slots:

name | 说明
|---|---|
default | 滚动的主体内容区域
pulldown | 下拉刷新的内容
pullup |  上拉加载的内容

### Methods:
方法名 | 说明 | 参数
|---|---|---|
initScroll | 初始化scroll组件 | |
disable | 禁用 better-scroll，DOM 事件（如 touchstart、touchmove、touchend）的回调函数不再响应 | |
enable | 启用 better-scroll, 默认 开启 | |
refresh | 重新计算 better-scroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常（当页面无法滚动时，可尝试调用此方法）| |
scrollTo | 滚动到指定的位置 | (scrollToX, scrollToY, scrollToTime, easing)接收4个参数，1.x横轴坐标(px) 2.y 纵轴坐标(px) 3.滚动动画执行的时长(ms) 4.easing 缓动函数，一般不建议修改
scrollToElement | 滚动到指定的目标元素 | (el, time, offsetX , offsetY )接收4个参数 详情请查看: [scrollToElement](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/api.html#scrolltoelementel-time-offsetx-offsety-easing)
destroy | 销毁 better-scroll，解绑事件
**forceUpdate** | 数据跟新后强制更新页面 | (dirty)接收1个 boolean 类型的参数，如果参数为true，说明还可以触发下拉或者上拉事件，若参数为false表示之后不可拉动，一般用于数据加载全部了

### Events:
事件名称 | 说明 | 回调参数
|---|---|---|
scroll | 触发时机：滚动过程中，具体时机取决于选项中的 probeType (触发事件在参数中需要开启 **listenScroll** ) | 共1个参数,类型Object, {x, y} 滚动的实时坐标
beforeScrollStart | 触发时机：滚动开始之前 (触发事件在参数中需要开启 **listenBeforeScroll** ) | 无
pullingDown | 触发时机：在一次下拉刷新的动作后，这个时机一般用来去后端请求数据。(触发事件在参数中需要开启 **pullDownRefresh** 相关配置 ) | 无
pullingUp | 触发时机：在一次上拉加载的动作后，这个时机一般用来去后端请求数据。(触发事件在参数中需要开启 **pullingUp** 相关配置 ) | 无


---
> 目前只提供了以上常用方法、Api,如有额外需要请 issue

# More detailed settings, please visit
[better-scroll document](https://ustbhuangyi.github.io/better-scroll/doc/)
