<template>
  <transition name="slide-right">
    <div class="content">
      <div class="content-wrapper" v-if="bookAvailable">
        <div class="content-item" v-for="(item, index) in navigation.toc" :key="index"
          @click="jumpTo(item.href)">
          <span class="text">{{item.label}}</span>
        </div>
      </div>
      <div class="empty" v-else>加载中...</div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    ifShowContent: Boolean,
    navigation: Object,
    bookAvailable: Boolean,
  },
  methods: {
    jumpTo(target) {
      this.$emit('jumpTo', target);
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/assets/styles/global.scss';
.content {
  position: absolute;
  z-index: 104;
  top: 0;
  left: 0;
  width: px2rem(400);
  height: 100%;
  background: #fff;
  font-size: px2rem(12);
  .content-wrapper {
    height: 100%;
    .content-item {
      margin: px2rem(20) 0;
      text-indent: px2rem(6);
      cursor: pointer;
    }
  }
}
</style>
