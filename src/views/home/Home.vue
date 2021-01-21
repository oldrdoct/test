<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行','新款','精选']"
                 @tabClick="tabClick"
                 ref="tabControl1"
                 class="tab-control"
                 v-show="isTabFlexd"/>
    <scroll class="content" ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
        <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
        <home-re-view :recommends="recommends"/>
        <home-fe-view/>
        <tab-control :titles="['流行','新款','精选']"
                     @tabClick="tabClick"
                     ref="tabControl2"/>
        <goods-list :goods="showGoods"/>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackUp"/>
  </div>

</template>

<script>
  import NavBar from "components/common/navbar/NavBar";
  import TabControl from "components/content/tabControl/TabControl";
  import GoodsList from "components/content/goods/GoodsList";
  import Scroll from "components/common/scroll/Scroll";
  import BackTop from "components/content/backTop/BackTop";

  import HomeSwiper from "./childComps/HomeSwiper";
  import HomeReView from "./childComps/HomeReView";
  import HomeFeView from "./childComps/HomeFeView";


  import {getHomeMultidata,getHomeGoods} from "network/home";
  import {debounce} from "common/utils";


  export default {
    name: "Home",
    components: {
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,
      HomeSwiper,
      HomeReView,
      HomeFeView
    },
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': { page: 0 , list: []},
          'new': { page: 0 , list: []},
          'sell': { page: 0 , list: []}
        },
        currentType: 'pop',
        isShowBackUp: false,
        tabOffsetTop: 0,
        isTabFlexd: false,
        saveY: 0
      }
    },
    created() {
    //      1.请求多个数据
      this.getHomeMultiData()
    //    2.请求商品数据
      this.getHomeGood('pop')
      this.getHomeGood('new')
      this.getHomeGood('sell')
    },
    activated() {
      this.$refs.scroll.scrollTo(0,this.saveY,0)
      this.$refs.scroll.refresh()
    },
    deactivated() {
      this.saveY = this.$refs.scroll.getScrollY()
    },
    mounted() {
      //3.监听iten中的图片
      const refresh = debounce(this.$refs.scroll.refresh,200)
      this.$bus.$on('itemImageLoad', () =>{
        refresh()
      })
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
    methods: {
      /**
       * 事件监听相关方法
       */
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
         }
         this.$refs.tabControl1.currentIndex = index
        this.$refs.tabControl2.currentIndex = index
       },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      contentScroll(position) {
        //1.判断backtop 是否显示
        this.isShowBackUp = (-position.y) > 1000
      //  2.判断tabcontrol是否有flex属性
        this.isTabFlexd = (-position.y) > this.tabOffsetTop
      },
      loadMore() {
        this.getHomeGood(this.currentType)
      },
      swiperImageLoad() {
        this.tabOffsetTop =  this.$refs.tabControl2.$el.offsetTop
      },
      /**
       * 网络请求相关方法
       */
      getHomeMultiData() {
        getHomeMultidata().then(res =>{
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        })
      },
      getHomeGood(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type,page).then(res =>{
          this.goods[type].list.push(...res.data.list);
          this.goods[type].page += 1;

          this.$refs.scroll.finishPullUp()
        })
      }
    }
  }
</script>

<style scoped>
  #home {
    height: 100vh;
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
  .tab-control {
    position: relative;
    z-index: 9;
  }
  .content {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
</style>
