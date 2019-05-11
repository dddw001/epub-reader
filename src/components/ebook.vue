<template>
  <div class="ebook">
    <title-bar :isTitleAndMenuShow="isTitleAndMenuShow" :currentTitle="currentTitle"></title-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <menu-bar
      :isTitleAndMenuShow="isTitleAndMenuShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      @setFontSize="setFontSize"
      :themeList="themeList"
      :defaultTheme="defaultTheme"
      @setTheme="setTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
      ref="menuBar">
    </menu-bar>
  </div>
</template>

<script>
import Epub from 'epubjs';
import titleBar from './title-bar';
import menuBar from './menu-bar';

const DOWNLOAD_URL = '/static/2018_Book_AgileProcessesInSoftwareEngine.epub';
export default {
  name: 'ebook',
  data() {
    return {
      isTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 },
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: 'default',
          style: {
            body: {
              color: '#000', background: '#fff',
            },
          },
        },
        {
          name: 'eye',
          style: {
            body: {
              color: '#000', background: '#ceeaba',
            },
          },
        },
        {
          name: 'night',
          style: {
            body: {
              color: '#fff', background: '#000',
            },
          },
        },
        {
          name: 'gold',
          style: {
            body: {
              color: '#000', background: 'rgb(241, 236, 129)',
            },
          },
        },
      ],
      defaultTheme: 0,
      bookAvailable: false,
      navigation: {},
      currentTitle: '',
    };
  },
  components: {
    titleBar,
    menuBar,
  },
  mounted() {
    this.showEpub();
  },
  methods: {
    // 电子书的解析和渲染
    showEpub() {
      // 生成Book
      this.book = new Epub(DOWNLOAD_URL);
      // 生成Rendition 通过renderTo方法，第一个参数是挂载的dom的id
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight,
      });
      // 通过Rendition.display渲染电子书
      this.rendition.display();

      // 获取Theme对象
      this.theme = this.rendition.themes;
      // 设置默认字体
      this.setFontSize(this.defaultFontSize);

      // this.theme.register(name, styles) 注册主题
      // this.theme.select(name) 根据主题名选择主题
      this.registerTheme();
      this.setTheme(this.defaultTheme);

      // 通过epubjs的钩子函数来获取Locations对象
      this.book.ready.then(() => this.book.locations.generate()).then(() => {
        this.navigation = this.book.navigation;
        this.locations = this.book.locations;
        this.bookAvailable = true;
      });
    },
    prevPage() {
      // Rendition.prev
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    nextPage() {
      // Rendition.next
      if (this.rendition) {
        this.rendition.next();
      }
      // 更新title
      this.updateTitle();
    },
    toggleTitleAndMenu() {
      this.currentTitle = '加载中...';
      this.updateTitle();
      this.isTitleAndMenuShow = !this.isTitleAndMenuShow;
      if (!this.isTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.theme) {
        this.theme.fontSize(fontSize + 'px');
      }
    },
    registerTheme() {
      this.themeList.forEach((theme) => {
        this.theme.register(theme.name, theme.style);
      });
    },
    setTheme(index) {
      this.defaultTheme = index;
      this.theme.select(this.themeList[index].name, this.themeList[index].style);
    },
    // progress进度条的数值 0-100
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    // 根据链接跳转到指定位置
    jumpTo(href) {
      this.rendition.display(href);
      this.hideTitleAndMenu();
    },
    hideTitleAndMenu() {
      // 隐藏标题栏和菜单栏
      this.isTitleAndMenuShow = false;
      // 隐藏菜单栏弹出的设置栏
      this.$refs.menuBar.hideSetting();
      // 隐藏目录
      this.$refs.menuBar.hideContent();
    },
    updateTitle() {
      if (this.isTitleAndMenuShow && this.navigation.toc) {
        this.navigation.toc.filter((item) => {
          if (item.href === this.rendition.location.start.href) {
            this.currentTitle = item.label;
          }
          return 0;
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/assets/styles/global.scss';
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
