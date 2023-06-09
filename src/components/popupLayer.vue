<template>
  <div
    class="pop-container"
    :class="{ show: visible, animation: isAnimation }"
    :style="{ zIndex }"
    :id="id"
  >
    <slot></slot>
  </div>
</template>
<script>
export default {
  name: "popupLayer",
  props: {
    // 显示状态，默认 false 不显示
    visible: Boolean,
    // 是否开启动画，默认开启 true
    isAnimation: {
      type: Boolean,
      default: true,
    },
    // 容器DOM的层级
    zIndex: {
      type: Number,
      default: 1000,
    },
    // 设置存储于locaStorage中的字段名称，默认 historyState
    storeName: {
      type: String,
      default: "historyState",
    },
  },
  data() {
    return {
      index: 0, //返回层级
      isSynced: false, // 是否已经同步过
      id: "popuplayer_" + this.guid(), //弹层唯一 id（容器唯一id）
    };
  },
  watch: {
    visible(val) {
      this[val ? "show" : "hide"]();
    },
  },
  mounted() {
    window.addEventListener("popstate", (e) => {
      // 浏览器历史状态  （控制台可以直接输入：history）
      let curState = e.state && e.state.id ? e.state.id : "";
      // 获取当前的 本地popuplayer历史记录
      let historyState = this.store().getCur();
      // 获取弹出层的层级数
      let _len = historyState.length;
      // 如果 弹出层的层级数 有值 并且 当前的返回事件id值 不等于
      if (_len && curState != historyState[_len - 1]) {
        if (historyState[_len - 1] == this.id) {
          // 抛出关闭事件
          this.$emit("onClose");
          // 关闭弹出层
          this.$emit("update:visible", false);
          // 删除弹出层id
          this.store().pop();
          // 点击物理返回键，直接触发 popstate 事件，走到这里 说明 上一步 已经同步删除了本地 弹出层id
          this.isSynced = true;
          // 输入框失去焦点
          document.activeElement.blur();
        }
      }
    });
  },
  created() {
    // 刷新页面 就重置
    this.store().reset();
    /**
     *  触发顺序说明：
     *  非物理返回键触发：先 只需 hide 再触发 物理返回事件 popstate
     *  物理返回，则 先触发 popstate 事件，再触发 hide，需要一个中间变量（isSynced）来调和
     */
  },
  methods: {
    /**
     * @description: 基于 localStorage 封装的组件所需的相关方法
     */
    store() {
      // 为保证无依赖，我们采用 localStorage
      const name = this.storeName;
      const store = localStorage;
      const parse = JSON.parse;
      const stringify = JSON.stringify;
      const old = parse(store.getItem(name));
      return {
        reset() {
          if (!store.getItem(name) || old.length) {
            store.setItem(name, stringify([]));
          }
        },
        update(newVal) {
          store.setItem(name, stringify([...old, newVal]));
        },
        getCur() {
          return old;
        },
        pop() {
          old.pop();
          store.setItem(name, stringify(old));
        },
        push(val) {
          store.setItem(name, stringify([...old, val]));
        },
      };
    },
    /**
     * @description: 为保证每个弹出层的id不同，生成guid的方法（不能用时间戳哈）
     * @param  {*}
     * @return {*}
     */
    guid() {
      return Array.from(
        { length: 8 },
        (_, i) =>
          (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1) +
          ([1, 2, 3, 4].includes(i) ? "-" : "")
      ).join("");
    },
    /**
     * @description: 打开弹层
     * 1. 打开弹出层
     * 2. 历史堆栈中 push 一个状态
     * 3. 往外抛出 打开弹出层 成功 后的 回调
     * 4. 本地自定义堆栈中同步 弹出层 id
     */
    show() {
      // 历史堆栈 push 一个 状态
      window.history.pushState({ id: this.id }, "");
      // 本地历史堆栈数字同步记录
      this.store().push(this.id);
      // 对外抛出弹出层打开事件
      this.$emit("onOpen", this.id);
    },
    /**
     * @description: 关闭弹出层
     * 判断用户是否直接点击物理返回键：
     * 1. 如果用户直接点击的是物理返回键，说明已经在   popstate 事件中已经同步删除 弹出层id了，此时这里无需操作
     * 2. 如果用户是通过点击按钮事件触发关闭（如：改变 <popupLayer :visible.sync="show" ><popupLayer> 的 show 的状态 false 或 true），
     *    则同步本地弹出层id数组（本地历史记录数组） 和 历史堆栈
     *
     */
    hide() {
      // 如果用户是通过点击物理返回键关闭，说明 历史栈 和 本地历史记录 已经同步过，此时只需还原 isDel 状态为 false 即可
      if (this.isSynced) return (this.isSynced = false);
      // 删除弹出层id
      this.store().pop();
      // 同步历史记录
      this.index ? history.back(-this.index) : history.back();
    },
    setLevel(level) {
      this.index = level;
      setTimeout(() => {
        this.index = 0;
      }, 300);
    },
  },
};
</script>

<style scoped>
.pop-container {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  width: 100%;
  backface-visibility: hidden;
  z-index: 10;
  transform: translateX(100%);
  background-color: #fff;
}
.pop-container.animation {
  transition: transform 0.3s;
}
.pop-container.show {
  transform: translateX(0);
}
</style>