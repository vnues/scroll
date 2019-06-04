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
