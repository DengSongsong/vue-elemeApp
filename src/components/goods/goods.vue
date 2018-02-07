<template>
  <div class="goods">
    <!-- 左侧菜单栏 -->
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="item.index" class="menu-item" :class="{'current':currentIdedx===index}" @click="selectMenu(index)">
          <span class="text">
            <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!-- 右侧商品展示 -->
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" :key="item.index" class="food-list" ref="foodList">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="item in item.foods" :key="item.index" class="food-item">
              <div class="icon">
                <img :src="item.icon" width="57" height="57"/>
              </div>
              <div class="content">
                <h2 class="name">{{item.name}}</h2>
                <p class="desc">{{item.description}}</p>
                <div class="extra">
                  <span class="count">月售{{item.sellCount}}</span>
                  <span class="praise">好评率{{item.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{item.price}}</span>
                  <span class="old" v-show="item.oldPrice">￥{{item.oldPrice}}</span>
                </div>
                <!-- 选购商品 -->
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="item"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <!-- 购物车 -->
    <shopcart :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice" ref="shopcart"></shopcart>
  </div>
</template>
<script>
import axios from 'axios'
import BScroll from 'better-scroll'
import shopcart from '@/components/shopcart/shopcart.vue';
import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
const ERR_OK = 0;
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return{
      goods: [],
      // 每一类型(li class="food-list")的所占长度(区间)存入该数组中
      listHeight: [],
      // 实时监控右侧栏滚动时y轴方向的位置，初始化为0
      scrollY: 0
    }
  },
  computed: {
    // 左侧栏菜单的索引与右侧栏的scrollY的值做映射，使之分类对应起来
    currentIdedx(){
      for(let i = 0; i < this.listHeight.length; i++){
        // 当前索引高度(当前的菜单分类的开始的高度)
        let height1 = this.listHeight[i];
        // 下一个索引高度(当前的菜单分类结束时的高度即下一个分类开始时的高度)
        let height2 = this.listHeight[i + 1];
        // 如果右侧栏滚动到了最下面(最后一个分类),height2不存在 || 当前滚动到(scrollY)某一个分类的区间中
        if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){
          // 返回当前菜单分类的索引值
          return i
        }
      }
      return 0;
    },
    // 被选中的商品
    selectFoods() {
      let foods = [];
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if(food.count){
            foods.push(food);
          }
        });
      });
      return foods;
    }
  },
  created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
     // 获取goods组件的数据
      axios.get('api/goods').then((response) => {
        response = response.data;
        // console.log(response);
        if(response.errno === ERR_OK){
          this.goods = response.data;
          // console.log(this.goods[0].foods[0].icon)

          // 数据更新及dom更新是异步的，如需要操作dom需要dom更新完成
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          })
        }
      })
  },
  methods: {
    // 菜单栏及商品栏产生滚动效果
    _initScroll() {
      // 菜单栏滚动
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        // better-scroll 默认会阻止浏览器的原生 click 事件。当设置为 true，better-scroll 会派发一个 click 事件，
        // 我们会给派发的 event 参数加一个私有属性 _constructed，值为 true。
        click: true
      });
      // console.log(this.$refs.menuWrapper);
      // 商品栏滚动
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        // 在屏幕滑动的过程中，
        // 且在 momentum 滚动动画运行过程中实时派发 scroll 事件
        // scroll事件：实时监控滚动时坐标
        probeType: 3,
        click: true
      });
      // 右侧栏派发scroll事件
      this.foodsScroll.on('scroll', (pos) => {
        // 判断滑动方向，避免下拉时分类高亮错误（如第一分类商品数量为1时，下拉使得第二分类高亮）
        if(pos.y <= 0){
          this.scrollY = Math.abs(Math.round(pos.y));
        }
      })

    },
    // 计算每个li(class="food-list")的长度，并存入数组listHeight中
    _calculateHeight() {
      // 获取全部的ref:foodList元素<li></li>
      // 数组
      let foodList = this.$refs.foodList;
      // console.log(foodList);
      let height = 0;
      this.listHeight.push(height);
      for(let i = 0; i < foodList.length; i++){
        let item = foodList[i];
        // console.log(item);
        height += item.clientHeight;
        // console.log(height);
        this.listHeight.push(height);
      }
      // console.log(this.listHeight);
    },
    // 左侧栏菜单点击
    selectMenu(index) {
      // console.log(index);
      let foodList = this.$refs.foodList;
      let el = foodList[index];
      // 滚动到指定的目标元素
      this.foodsScroll.scrollToElement(el, 300);
    },
    addFood(target){//参数target是从子组件cartcontrol传递过来的
      // console.log(target)
      
      // 拿到了想要触发小球下落(增加商品数量)的按钮元素，对其进行处理
      // 实现小球下落的动作
      this._drop(target);
    },
    // 调用shopcart组件中的drop()方法
    _drop(target) {
      this.$refs.shopcart.drop(target);
      // console.log(target);
      // console.log(this.$refs.shopcart.drop);
      // console.log(this.$refs.shopcart.drop());
    }
  },
  components: {
    shopcart,
    cartcontrol
  }
}
</script>
<style lang="stylus">
@import "../../common/stylus/mixin.styl"
  .goods
    display flex
    position absolute
    top 174px
    bottom 46px
    width 100%
    overflow: hidden
    .menu-wrapper
      flex 0 0 80px
      width 80px
      background #f3f5f7
      .menu-item
        display table
        height 54px
        width 56px
        line-height 14px
        padding 0 12px
        &.current
          position: relative
          z-index: 10
          margin-top: -1px
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display table-cell
          width 56px
          vertical-align middle
          text-align center
          font-size 12px
          border-1px(rgba(7, 17, 27, 0.1))
    .foods-wrapper
      flex 1
      .title
        padding-left: 14px
        height 26px
        font-size 12px
        line-height: 26px
        border-left: 2px solid #d9dde1
        color: rgb(147, 153, 159)
        background: #f3f5f7
      .food-item
        display flex
        margin 18px
        padding-bottom 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom 0
        .icon
          flex 0 0 57px
          margin-right 10px
          img
            border-radius 2px
        .content
          flex 1
          .name
            margin 2px 0 8px 0
            height 14px
            line-height 14px
            font-size 14px
            color: rgb(7, 17, 27)
          .desc
            line-height 10px
            font-size 12px
            color: rgb(147, 153, 159)
            margin-bottom 8px
          .extra
            font-size 0
            margin-bottom 8px
            color: rgb(147, 153, 159)
            .count
              font-size 14px
              margin-right 12px
            .praise
              font-size 14px
          .price
            line-height 24px
            font-weight 700
            .now
              margin-right 8px
              font-size 14px
              color: rgb(240, 20, 20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position absolute
            right 0
            bottom 12px
</style>
