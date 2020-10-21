<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物城</div>
    </nav-bar>
    <scroll
      class="content"
      ref="scroll"
      @scroll="contentScroll"
      @pullingUp="loadMore"
      :probe-type="3"
      :pullUpLoad="true"
    >
      <home-swiper :banners="banners"></home-swiper>
      <home-recommend :recommends="recommends"></home-recommend>
      <home-feature></home-feature>
      <tab-control
        class="tab-control"
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick"
      ></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBack"></back-top>
  </div>
</template>

<script>
import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";

import HomeSwiper from "./childComps/HomeSwiper";
import HomeRecommend from "./childComps/HomeRecommend";
import HomeFeature from "./childComps/HomeFeature";

import { getHomeMultidata, getGoodsInfo } from "network/home";

export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    HomeRecommend,
    HomeFeature,
    TabControl,
    GoodsList,
    Scroll,
    BackTop,
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      currentType: "pop",
      isShowBack: false,
    };
  },
  created() {
    this.getHomeMultidata();
    this.getGoodsInfo("pop");
    this.getGoodsInfo("new");
    this.getGoodsInfo("sell");
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    },
  },
  methods: {
    //设置上拉加载,调用getGoodsInfo函数加载更多数据
    loadMore() {
      this.getGoodsInfo(this.currentType);
    },
    //BackTop设置隐藏显示
    contentScroll(position) {
      this.isShowBack = -position.y > 1000;
    },
    //设置BackTop到点击回到X0 Y0 位置
    backClick() {
      this.$refs.scroll.scroll.scrollTo(0, 0, 300);
    },
    //监听点击相关代码
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
      }
    },

    //  网络请求相关代码
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    getGoodsInfo(type) {
      const page = this.goods[type].page + 1;
      getGoodsInfo(type, page).then((res) => {
        this.goods[type].list.push(...res.data.list);
        // console.log(...res.data.list)
        this.goods[type].page += 1;
        //结束上拉加载行为
        this.$refs.scroll.scroll.finishPullUp();
      });
    },
  },
};
</script>
<style scoped>
#home {
  height: 100vh;
  position: relative;
}
.home-nav {
  background-color: var(--color-tint);
  color: #fff;
  z-index: 9;
}
.tab-control {
  position: sticky;
  top: 44px;
}
.content {
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
  overflow: hidden;
}
</style>
