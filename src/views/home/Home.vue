<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">蘑菇鸡</div>
    </nav-bar>
    <home-swiper :banners="banners"></home-swiper>
    <home-recommend-view :recommend="recommend"></home-recommend-view>
    <home-feature-view></home-feature-view>
    <tab-control class="tab-control" :titles="['流行', '新款', '精选']" @tabClick="tabClick"></tab-control>
    <goods-list :goods="showGoods"></goods-list>
  </div>
</template>

<script>
// 子组件
import HomeSwiper from "./childComponents/HomeSwiper";
import HomeRecommendView from "./childComponents/HomeRecommendView";
import HomeFeatureView from "./childComponents/HomeFeatureView";
import GoodsList from "components/content/goods/GoodsList";

// 公共组件
import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";

// 方法
import {getHomeMultidata, getHomeGoods} from "network/home"


export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    HomeRecommendView,
    HomeFeatureView,
    TabControl,
    GoodsList
  },
  data() {
    return {
      banners: [],
      recommend: [],
      currentType: 'pop',
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []},
      }
    }
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    }
  },
  created() {
    // 1.请求多个数据
    this.getHomeMultidata()
    
    // 2.请求商品列表页（流行、最新、精选）
    this.getHomeGoods('pop')
    this.getHomeGoods('new')
    this.getHomeGoods('sell')
  },
  methods: {
    /*
    * 网络请求相关的方法
    * */
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        // console.log('getHomeMultidata', res)
        this.banners = res.data.banner.list
        this.recommend = res.data.recommend.list
      })
    },
    
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        // this.goods[type].list.concat(res.data.list)  // 数组拼接
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1
      })
    },
    /*
    * 事件监听相关的方法
    * */
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = 'pop'
              break
        case 1:
          this.currentType = 'new'
          break
        case 2:
          this.currentType = 'sell'
          break
      }
    }
  }
}
</script>

<style scoped>
#home {
  padding-top: 44px;
}

.home-nav {
  background-color: var(--color-tint);
  color: white;
  
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9;
}

.tab-control {
  position: sticky;
  top: 44px;
  z-index: 9;
}
</style>
