<template>
  <div class="dsf-muti-ellipsis" ref="container">
    <div
      class="ellipsis-webkit-type"
      :style="!isComplate && webkitStyle"
      v-if="!clientIsIe"
    >
      <div :style="txtStylePlus">{{ text }}</div>
    </div>
    <div class="ellipsis-ie-type" v-else>
      <div class="dsf-muti-ellipsis-content">
        <span ref="text" :style="txtStylePlus">{{ textFat.viewText }}</span>
        <transition name="ellipsis-slide">
          <span class="more" :style="txtStylePlus" v-if="isComplate">{{
            textFat.moreText
          }}</span>
        </transition>
      </div>
    </div>
  </div>
</template>
<script>
let clientIsIe = "";
export default {
  name: "DsfMutiEllipsis",
  props: {
    // 需要展示的内容
    text: {
      type: String,
      default: "",
    },
    // // 展示区域最大高度（优先级更高，自动匹配最合适的行数） ====== 抛弃--保持跟webkit多行省略一致
    // height: Number,
    // // 只有设置高度属性时才生效。 ====== 抛弃--保持跟webkit多行省略一致
    // heightType: {
    //   type: String,
    //   default: "remove", // 当设置高度不等于行高的倍数时，remove -> 采用此高度下最大的整数行数  / increase -> 采用此高度下最大的整数行数 + 1
    // },
    // 展示区域做大行数
    maxLine: {
      type: Number,
      default: 1,
    },
    // 是否完整展示内容
    isComplate: {
      type: Boolean,
      default: false,
    },
    // 字体样式
    txtStyle: Object,
    // 超出部分内容 展示字符
    more: {
      type: String,
      default: "...",
    },
    // 手动调整距离
    tabs: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      textFat: {
        viewText: "", // 展示区域文字
        moreText: "", // 省略文字
      },

      containerStyle: {
        width: 0,
      },
      watchDisplay: "",
      textStyle: {
        height: 0,
        letterSpacing: 0,
        lineHeight: 0,
      },
      prDisplayNode: null, // 最近的祖先元素被隐藏的节点
      clientIsIe: "",
      webkitStyle: {},
      // 消除tab标签之间幽灵元素
      txtStylePlus: Object.assign({}, { "font-size": "16px" }, this.txtStyle),
    };
  },
  watch: {
    text() {
      this.getStylesOfText();
    },
    isComplate(value) {
      if (this.textFat.moreText) {
        if (value) {
          const l = this.textFat.viewText.length - this.$props.more.length;
          this.textFat.viewText = this.textFat.viewText.substring(0, l);
        } else {
          this.textFat.viewText = this.textFat.viewText + this.$props.more;
        }
      }
    },
  },
  computed: {
    limitWidth() {
      // let { heightType, height, maxLine } = this.$props;
      // if (height !== undefined) {
      //   let numberLineHeight =
      //     this.textStyle.lineHeight === "normal"
      //       ? 1.2 * this.txtStyle.height
      //       : parseFloat(this.textStyle.lineHeight);
      //   maxWidth = ~~(height / numberLineHeight) * this.containerStyle.width;
      //   if (heightType === "increase" && height % numberLineHeight) {
      //     maxWidth += this.containerStyle.width;
      //   }
      // } else {
      //   maxWidth = maxLine * this.containerStyle.width;
      // }
      return this.maxLine * this.containerStyle.width;
    },
  },
  mounted() {
    let userAgent = navigator.userAgent;
    if (clientIsIe !== "") {
      this.clientIsIe = clientIsIe;
      if (clientIsIe) {
        this.getStylesOfText();
      }
      return;
    }
    if (
      (userAgent.indexOf("compatible") > -1 &&
        userAgent.indexOf("MSIE") > -1) ||
      userAgent.indexOf("Trident") > -1
    ) {
      clientIsIe = true;
      this.getStylesOfText();
    } else {
      clientIsIe = false;
      this.webkitStyle = {
        display: "-webkit-box",
        "-webkit-box-orient": "vertical",
        overflow: "hidden",
        "-webkit-line-clamp": this.maxLine,
        "text-overflow": "ellipsis",
      };
    }
    this.clientIsIe = clientIsIe;
  },
  methods: {
    getStylesOfText() {
      this.$nextTick(() => {
        setTimeout(() => {
          if (!this.$refs.container.offsetWidth) {
            this.watchDisplayNone();
            return;
          }
          // 获取填装text容器总宽度
          let span = this.$refs.text.cloneNode();
          span.innerHTML = this.$props.text;
          for (let k in this.txtStylePlus) {
            span.style[k] = this.txtStylePlus[k];
          }
          span.style.opacity = 0;
          this.$refs.container.appendChild(span);
          // 展示字体样式
          let textStyle = null;
          if (window.getComputedStyle) {
            textStyle = window.getComputedStyle(this.$refs.text);
            this.textStyle = {
              height: +textStyle.fontSize.replace(/px/, ""),
              letterSpacing: +textStyle.letterSpacing.replace(/px/, "") || 0,
              lineHeight: textStyle.lineHeight,
            };
          } else {
            textStyle = this.$refs.text.currentStyle();
          }
          this.containerStyle.width = Math.max(
            span.offsetWidth,
            this.$refs.container.offsetWidth -
              this.textStyle.height -
              this.textStyle.letterSpacing
          );
          this.$refs.container.removeChild(span);
          this.getChatsLength();
        }, 50);
      });
    },
    getChatsLength() {
      let { more, text, tabs } = this.$props;
      // 规定区域内字符串个数
      let n = ~~Math.max(
        this.limitWidth /
          (this.textStyle.height + this.textStyle.letterSpacing) -
          more.length,
        0
      );
      const canvasContext = document.createElement("canvas").getContext("2d");
      let textStyle = "";
      if (window.getComputedStyle) {
        textStyle = window.getComputedStyle(this.$refs.text);
      } else {
        textStyle = this.$refs.text.currentStyle;
      }
      let textFontStyle =
        textStyle.font ||
        `${textStyle.fontWeight} ${
          textStyle.fontSize
        } ${textStyle.fontFamily.replace(/-apple-system,/g, "")}`;
      canvasContext.font = textFontStyle;
      let relStrWidth = 0;
      if (
        canvasContext.measureText(this.text).width +
          (this.text.length - 1) * this.textStyle.letterSpacing <
        this.limitWidth
      ) {
        this.textFat.viewText = this.text;
      } else {
        let moreLength =
          canvasContext.measureText(more).width +
          more.length * this.textStyle.letterSpacing;
        while (n < this.text.length) {
          // 中文字符大小跟fontSize一致，英文/数字/英文符号等字符大小 小于fontsize
          let txt = text.substring(0, n);
          relStrWidth =
            canvasContext.measureText(txt).width +
            n * this.textStyle.letterSpacing +
            moreLength;
          if (relStrWidth > this.limitWidth) {
            break;
          }
          n++;
        }
        let splitIndex = n - 1 + tabs;
        this.textFat = {
          viewText: this.text.substring(0, splitIndex) + more,
          moreText: this.text.substring(splitIndex),
        };
      }
    },
    watchDisplayNone() {
      try {
        let parentDom = this.$refs.container.parentElement;
        let watchedObject = null;
        const _this = this;
        // 寻找最近的display为none的祖先元素
        while (parentDom) {
          if (parentDom.style.display !== "none") {
            parentDom = parentDom.parentElement;
          } else {
            watchedObject = parentDom.style;
            break;
          }
        }
        this.watchDisplay = watchedObject.display;
        const isWatched = Object.getOwnPropertyDescriptor(
          watchedObject,
          "display"
        );
        this.prDisplayNode = parentDom;
        let watchers = parentDom.EllipsisWatchers || {};
        if (!watchers[this._uid]) {
          // 注册订阅者
          const fn = function (value) {
            if (value !== "none" && !this.textFat.viewText) {
              this.getStylesOfText();
            }
          };
          watchers[this._uid] = fn.bind(this);
          parentDom.EllipsisWatchers = watchers;
        }
        // 若该元素之前就被代理过，则忽略
        if (!isWatched || !typeof isWatched.isWatched === "function") {
          Object.defineProperty(watchedObject, "display", {
            configurable: true,
            set: function (value) {
              let watchers = parentDom.EllipsisWatchers;
              // 更改Dom样式
              let style = this.cssText.replace(/display:.+/g, "");
              if (value.trim() !== "") {
                style += `display: ${value}`;
              }
              parentDom.setAttribute("style", style);
              _this.watchDisplay = value;

              // 分发当前节点下的所有组件
              for (let uid in watchers) {
                watchers[uid](value);
              }
            },
            get: function () {
              return _this.watchDisplay;
            },
          });
        }
      } catch (e) {
        console.log(e);
      }
    },
  },
  beforeDestroy() {
    if (this.prDisplayNode) {
      let watchers = this.prDisplayNode?.EllipsisWatchers;
      if (watchers) {
        delete watchers[this._uid];
      }
    }
  },
};
</script>
<style lang="css" scoped>
.dsf-muti-ellipsis-content {
  font-size: 0;
}
.ellipsis-slide-enter-active {
  transition: opacity 0.3s;
}
.ellipsis-slide-leave-active {
  transition: opacity 0.3s;
}
.ellipsis-slide-enter,
.ellipsis-slide-leave-to {
  opacity: 0;
}
.dsf-muti-ellipsis span {
  word-break: break-all;
}
</style>