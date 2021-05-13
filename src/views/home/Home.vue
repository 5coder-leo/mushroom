<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">蘑菇鸡</div>
    </nav-bar>
    <tab-control :titles="['流行', '新款', '精选']" @tabClick="tabClick" ref="tabControl1" class="tabControl"
                 v-show="isTabFixed"></tab-control>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <home-recommend-view :recommend="recommend"></home-recommend-view>
      <home-feature-view></home-feature-view>
      <tab-control :titles="['流行', '新款', '精选']" @tabClick="tabClick" ref="tabControl2"></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
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
import Scroll from "components/common/scroll/Scroll";
import BackTop from 'components/content/backTop/BackTop'


// 方法
import {getHomeMultidata, getHomeGoods} from "network/home"
import {debounce} from "common/utils";


export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    HomeRecommendView,
    HomeFeatureView,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
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
      },
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
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
  mounted() {
    const refresh = debounce(this.$refs.scroll && this.$refs.scroll.refresh, 100)
    // 3.图片加载完成的事件监听$bus
    this.$bus.$on('itemImageLoad', () => {
      refresh()
    })
  },
  destroyed() {
    console.log('home destroy');
  },
  activated() {
    this.$refs.scroll.refresh()
    this.$refs.scroll.scrollTo(0, this.saveY, 0)
  },
  deactivated() {
    this.saveY = this.$refs.scroll.getScrollY()
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
        
        this.$refs.scroll.finishPullUp()
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
      this.$refs.tabControl1.currentIndex = index
      this.$refs.tabControl2.currentIndex = index
    },
    contentScroll(position) {
      // 1.判断backTop是否显示
      this.isShowBackTop = (-position.y) > 1000
      // 2.决定tabControl是否吸顶
      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0)
    },
    loadMore() {
      this.getHomeGoods(this.currentType)
    },
    swiperImageLoad() {
      // 4.获取tabControl的offsetTop
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
    }
    
  }
}
</script>

<style scoped>
#home {
  /*padding-top: 44px;*/
  height: 100vh;
  position: relative;
}

.home-nav {
  background-color: var(--color-tint);
  color: #fff;
  
  /*position: fixed;*/
  /*left: 0;*/
  /*right: 0;*/
  /*top: 0;*/
  /*z-index: 9;*/
}

.content {
  overflow: hidden;
  
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}

.tabControl {
  position: relative;
  z-index: 9;
}
</style>
