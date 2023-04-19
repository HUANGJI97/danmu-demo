<template>
  <div class="header">
    <div class="title">这是一个展会<span class="tag">未开始</span></div>
    <div class="date">2023年04月16日13:02:40 - 2023年04月16日13:02:43</div>
    <vue-danmaku
      v-model="danmus"
      useSlot
      :channels="2"
      :speeds="100"
      loop
      style="height: 100px; width: 100%; margin-top: 16px"
    >
      <template slot="dm" slot-scope="{ index, danmu }">
        <div class="danmu-item" :key="index">{{ danmu }}</div>
      </template>
    </vue-danmaku>
  </div>
</template>

<script>
import vueDanmaku from "vue-danmaku";
export default {
  components: { vueDanmaku },
  props: {},
  data() {
    return {
      show: false,
      show1: false,
      danmus: [
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
        "丹棱縣成員水果經營部企業發佈了春見粑粑柑產品",
      ],
    };
  },
  methods: {},
};
</script>
<style>
.header {
  background: linear-gradient(#fef0e1, #fdf7ee);
  padding: 16px;
  /* background-color: aquamarine; */
}
.title {
  font-size: 18px;
  font-weight: bold;
  display: flex;
  align-items: center;
}
.tag {
  font-size: 12px;
  color: #a56b47;
  background: white;
  margin-left: 14px;
  padding: 0xp 5px;
}
.date {
  margin-top: 16px;
  font-size: 10px;
}
.danmu-item {
  background: red;
  font-size: 12px;
  color: #7f8494;
  background: #f1f1f6;
  border-radius: 10px;
  padding: 2px 10px;
  margin-bottom: 12px;
}
body {
  padding: 0px;
  margin: 0px;
}
.pop1 {
  height: 100%;
  background: brown;
}
</style>
