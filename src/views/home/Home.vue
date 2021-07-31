<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" />
      <recommend-view :recommends="recommends" />
      <FeatureView />
      <TabControl class="tab-control" :titles="['流行','新款','精品']" @tabClick="tabClick"/>
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
  import HomeSwiper from "./childComps/HomeSwiper";
  import RecommendView from "./childComps/RecommendView";
  import FeatureView from "./childComps/FeatureView";

  import NavBar from "components/common/navbar/NavBar";
  import TabControl from "components/content/tabControl/TabControl";
  import GoodsList from "components/content/goods/GoodsList";
  import Scroll from "components/common/scroll/Scroll";
  import BackTop from "components/content/backTop/BackTop";

  import {getHomeMultidata,getHomeGoods} from "network/home";

  export default {
    name: "home",
    components: {
      HomeSwiper,
      RecommendView,
      FeatureView,
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop
    },
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0,list:[]},
          'new': {page: 0,list:[]},
          'sell': {page: 0,list:[]}
        },
        currentType: 'pop',
        isShowBackTop:false
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list;
      }
    },
    created() {
      //1、请求多个数据
      this.getHomeMultidata();
      //2、请求商品数据
      this.getHomeGoods('pop');
      this.getHomeGoods('new');
      this.getHomeGoods('sell');
    },
    methods: {
      /**
       * 事件监听相关方法
       */
      tabClick(index) {
        console.log(index)
        switch(index){
          case 0:
            this.currentType = 'pop'
            break;
          case 1:
            this.currentType = 'new'
            break;
          case 2:
            this.currentType = 'sell'
            break;
        }

      },
      backClick() {
        this.$refs.scroll.scrollTop(0,0);
      },
      contentScroll(position) {
        this.isShowBackTop = (-position.y) > 1000
      },
      loadMore() {
        console.log('上拉加载更多');
        this.getHomeGoods(this.currentType)
        this.$refs.scroll.refresh()
      },
      /**
       * 网络请求相关代码
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        })
      },
      getHomeGoods(type) {
        let page = this.goods[type].page + 1;
        var that = this;
        getHomeGoods(type,page).then(res => {
          that.goods[type].list.push(...res.data.list);//数组解构赋值
          that.goods[type].page += 1;
          this.$refs.scroll.finishPullUp()
        });
      }
    }
  }
</script>
<!--style  标签后面的scoped表示：style标签里面定义的样式，只针对当前这个vue组件，其他组件不受影响（包括子组件也不受影响）-->
<style scoped>
  #home{
    padding-top: 44px;
    height: 100vh;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

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

  .content {
    /*height: 300px;*//*新大陆：如果这里设置了高度，则top和bottom两个属性只有top会生效，bottom不会生效；如果不设置heigh属性，则 top和bottom两个元素都会生效（这样就能够改变元素高度了）。牛逼！！！ */
    overflow: hidden;

    position: absolute;
    top:44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
  /*.content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 51px;
  }*/
</style>
