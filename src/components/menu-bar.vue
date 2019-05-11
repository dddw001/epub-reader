<template>
  <div class="menu-bar">
    <transition name="slide-up">
      <div>
        <div class="menu-wrapper" :class="{'hide-box-shadow': ifSettingShow}"
          v-show="isTitleAndMenuShow">
          <div class="icon-wrapper">
            <span class="icon-menu icon" @click="showSetting(3)"></span>
          </div>
          <div class="icon-wrapper">
            <span class="icon-progress icon" @click="showSetting(2)"></span>
          </div>
          <div class="icon-wrapper">
            <span class="icon-bright icon" @click="showSetting(1)"></span>
          </div>
          <div class="icon-wrapper">
            <span class="icon-share icon" @click="showSetting(0)"></span>
          </div>
        </div>
        <transition name="slide-up">
          <div class="setting-wrapper" v-show="ifSettingShow">
            <div class="setting-font-size" v-if="showTag === 0">
              <div class="preview" :style="{fontSize: fontSizeList[0].fontSize + 'px'}">A</div>
              <div class="select">
                <div class="select-wrapper" @click="setFontSize(item.fontSize)"
                  v-for="(item, index) in fontSizeList" :key="index">
                  <div class="line"></div>
                  <div class="point-wrapper">
                    <div class="point" v-show="defaultFontSize == item.fontSize">
                      <div class="small-point"></div>
                    </div>
                  </div>
                  <div class="line"></div>
                </div>
              </div>
              <div class="preview"
                :style="{fontSize: fontSizeList[fontSizeList.length - 1].fontSize + 'px'}">A</div>
            </div>
            <div class="setting-theme" v-else-if="showTag === 1">
              <div class="setting-theme-item" v-for="(item, index) in themeList" :key="index"
                @click="setTheme(index)">
                <div class="preview" :style="{background: item.style.body.background}"
                  :class="{'no-border': item.style.body.background !== '#fff'}"></div>
                <div class="text" :class="{'selected': index === defaultTheme}">{{item.name}}</div>
              </div>
            </div>
            <div class="setting-progress" v-else-if="showTag === 2">
              <div class="progress-wrapper">
                <input type="range" class="progress" max="100" min="0" step="1"
                  :value="progress" :disabled="!bookAvailable" ref="progress"
                  @change="onProgressChange($event.target.value)"
                  @input="onProgressInput($event.target.value)">
              </div>
              <div class="text-wrapper">
                <span>{{bookAvailable ? progress + '%' : '加载中...'}}</span>
              </div>
            </div>
          </div>
        </transition>
      </div>
    </transition>
    <content-view
      :ifShowContent="ifShowContent"
      v-show="ifShowContent"
      :navigation="navigation"
      :bookAvailable="bookAvailable"
      @jumpTo="jumpTo">
    </content-view>
    <transition name="fade">
      <div class="content-mask" v-show="ifShowContent" @click="hideContent">
      </div>
    </transition>
  </div>
</template>

<script>
import ContentView from '@/components/content';

export default {
  name: 'menuBar',
  components: {
    ContentView,
  },
  props: {
    isTitleAndMenuShow: {
      type: Boolean,
      default: false,
    },
    fontSizeList: {
      type: Array,
      default: () => [],
    },
    defaultFontSize: {
      type: Number,
      default: 16,
    },
    themeList: Array,
    defaultTheme: Number,
    bookAvailable: Boolean,
    navigation: Object,
  },
  data() {
    return {
      ifSettingShow: false,
      showTag: 0,
      progress: 0,
      ifShowContent: false,
    };
  },
  methods: {
    showSetting(tag) {
      this.showTag = tag;
      if (this.showTag === 3) {
        this.ifShowContent = true;
        this.ifSettingShow = false;
      } else {
        this.ifSettingShow = true;
      }
    },
    hideSetting() {
      this.ifSettingShow = false;
    },
    setFontSize(fontSize) {
      this.$emit('setFontSize', fontSize);
    },
    setTheme(index) {
      this.$emit('setTheme', index);
    },
    // 拖动过程不触发，拖动结束时触发，点击触发
    onProgressChange(val) {
      this.$emit('onProgressChange', val);
    },
    // 连续拖动和点击都触发
    onProgressInput(val) {
      this.progress = val;
      this.$refs.progress.style.backgroundSize = `${this.progress}% 100%`;
    },
    hideContent() {
      this.ifShowContent = false;
    },
    jumpTo(target) {
      this.$emit('jumpTo', target);
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/assets/styles/global.scss';
.menu-bar {
  .menu-wrapper {
    position: absolute;
    bottom: 0;
    left: 0;
    z-index: 101;
    display: flex;
    width: 100%;
    height: px2rem(48);
    background: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    &.hide-box-shadow {
      box-shadow: none;
    }
    .icon-wrapper {
      flex: 1;
      @include center;
      cursor: pointer;
      .icon-progress {
        font-size: px2rem(24);
      }
      .icon-bright {
        font-size: px2rem(28);
      }
    }
  }
  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    z-index: 101;
    width: 100%;
    height: px2rem(60);
    background: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    .setting-font-size {
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 px2rem(40);
        @include center;
      }
      .select {
        display: flex;
        flex: 1;
        .select-wrapper {
          flex: 1;
          display: flex;
          align-items: center;
          &:first-child {
            .line:first-child {
              border-top: none;
            }
          }
          &:last-child {
            .line:last-child {
              border-top: none;
            }
          }
          .line {
            flex: 1;
            height: 0;
            border-top: px2rem(1) solid #ccc;
          }
          .point-wrapper {
            position: relative;
            flex: 0 0 0 ;
            width: 0;
            height: px2rem(7);
            border-left: px2rem(1) solid #ccc;
            cursor: pointer;
            .point {
              position: absolute;
              top: px2rem(-8);
              left: px2rem(-10);
              width: px2rem(20);
              height: px2rem(20);
              @include center;
              border-radius: 50%;
              background: white;
              border: px2rem(1) solid #ccc;
              box-shadow: 0 px2rem(4) px2rem(4) rgba(0, 0, 0, 0.15);
              .small-point {
                width: px2rem(5);
                height: px2rem(5);
                border-radius: 50%;
                background: #000;
              }
            }
          }
        }
      }
    }
    .setting-theme {
      display: flex;
      height: 100%;
      .setting-theme-item {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: px2rem(5);
        box-sizing: border-box;
        cursor: pointer;
        .preview {
          flex: 1;
          border: 1px solid #ccc;
          box-sizing: border-box;
          &.no-border {
            border: none;
          }
        }
        .text {
          flex: 0 0 px2rem(20);
          font-size: px2rem(14);
          color: #ccc;
          @include center;
          &.selected {
            color: #333;
          }
        }
      }
    }
    .setting-progress {
      width: 100%;
      height: 100%;
      .progress-wrapper {
        width: 100%;
        height: 70%;
        @include center;
        padding: 0 px2rem(30);
        box-sizing: border-box;
        .progress {
          width: 100%;
          -webkit-appearance: none;
          height: px2rem(2);
          background: -webkit-linear-gradient(#999, #999) no-repeat, #ddd;
          background-size: 0 100%;
          &:focus {
            outline: none;
          }
          // 拖动的手柄
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: px2rem(20);
            width: px2rem(20);
            border-radius: 50%;
            background: white;
            box-shadow: 0 4px 4px 0 rgba(0, 0, 0, 0.15);
            border: px2rem(1) solid #ddd;
          }
        }
      }
      .text-wrapper {
        text-align: center;
        font-size: px2rem(14);
      }
    }
  }
  .content-mask {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 101;
    display: flex;
    width: 100%;
    height: 100%;
    background: rgba(51, 51, 51, 0.8);
  }
}
</style>
