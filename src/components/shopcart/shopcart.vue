<template>
  <div>
    <div class="shopcart">
      <!-- 购物车栏 -->
      <div class="content" @click="toggleList">
        <div class="content-left">
            <div class="logo-wrapper">
              <div class="logo" :class="{'highlight':totalCount>0}">
                <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
              </div>
              <div class="num" v-show="totalCount>0">{{totalCount}}</div>
            </div>
            <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
            <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right" @click.stop.prevent="pay">
            <div class="pay" :class="payClass">{{payDesc}}</div>
        </div>
      </div>
      <!-- 增加数量购物车小球下落效果 -->
      <div class="ball-container">
        <div v-for="item in balls" :key="item.index">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="item.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <!-- 购物车栏点击被选择商品列表 -->
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="item in selectFoods" :key="item.index">
                <span class="name">{{item.name}}</span>
                <div class="price">
                  <span>￥{{item.price * item.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="item"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <!-- 购物车栏点击弹窗 -->
    <transition name="fade">
      <div class="list-mask" @click="hideList" v-show="listShow"></div>
    </transition>
  </div>
</template>
<script>
import BScroll from 'better-scroll';
import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
export default {
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [
          {
            price: 10,
            count: 1
          }
        ]
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      fold: true,
      dropBalls: []
    }
  },
  computed: {
    // 总价
    totalPrice() {
      let total = 0;
      this.selectFoods.forEach((food) => {
        total += food.price * food.count;
      });
      return total;
    },
    // 总数量
    totalCount() {
      let count = 0;
      this.selectFoods.forEach((food) => {
        count += food.count;
      });
      return count;
    },
    // 结算栏
    payDesc() {
      if(this.totalPrice === 0) {
        return `￥${this.minPrice}起送`;
      }else if(this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice;
        return `还差￥${diff}起送`
      }else {
        return '去结算';
      }
    },
    payClass() {
      if(this.totalPrice < this.minPrice) {
        return 'not-enough';
      }else {
        return 'enough';
      }
    },
    listShow() {
      // 被选择商品个数为0，折叠
      if(!this.totalCount) {
        this.food = true;
        return false;
      }
      let show = !this.fold;
      if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.listContent, {
                click: true
              });
            } else {
              // 重新计算 better-scroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常
              this.scroll.refresh();
            }
          });
        }
      return show;
    }
  },
  methods: {
    // 弹出及隐藏被选择的商品列表
    toggleList() {
      if(!this.totalCount){
        return;
      }
      this.fold = !this.fold;
    },
    // 隐藏被选择的商品列表
    hideList() {
      this.fold = true;
    },
    // 清空被选择的商品列表
    empty() {
      this.selectFoods.forEach((food) => {
        food.count = 0;
      });
    },
    // 支付
    pay() {
      if (this.totalPrice < this.minPrice) {
        return;
      }
      window.alert(`支付${this.totalPrice}元`);
      this.empty();
    },
    addFood(target) {
      this.drop(target);
    },
    drop(el){
      // console.log(el);
      for(let i =0; i < this.balls.length; i++){
        let ball = this.balls[i];
        // 当ball.show为false时就取出来，并设为true和用一个属性el保留element
        if(!ball.show){
          ball.show = true;
          ball.el = el;
          // 将可以下落的小球存入数组
          this.dropBalls.push(ball);
          // 找到可以下落的小球就可以退出循环
          return;
        }  
      }
    },
    beforeDrop(el) {
      // console.log(el);
      let count = this.balls.length;
      while(count--) {
        let ball = this.balls[count];
        // console.log(ball);
        if(ball.show){
          // 返回值是一个 DOMRect 对象,包含了一组用于描述边框的只读属性——left、top、right和bottom，单位为像素。
          // 除了 width 和 height 外的属性都是相对于视口的左上角位置而言
          let rect = ball.el.getBoundingClientRect();
          // console.log(rect);
          // 增加按钮与购物车x轴方向间的距离
          let x = rect.left - 32;
          // 绝对值是增加按钮与购物车y轴方向间的距离
          let y = -(window.innerHeight - rect.top - 22);
          // 还未触发前将小球的状态设置为空
          el.style.display = '';
          // 为触发前将小球的位置移到与增加按钮相同的位置
          // 外层元素做纵向上的变化
          el.style.webkitTransform = `translate3d(0, ${y}px, 0)`;
          el.style.Transform = `translate3d(0, ${y}px, 0)`;
          // 内层元素做横向向上的变化
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
          inner.style.Transform = `translate3d(${x}px, 0, 0)`;
        }
      }
    },
    dropping(el, done) {
      // 浏览器重置
      let rf = el.offsetHeight;
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(0,0,0)';
          inner.style.transform = 'translate3d(0,0,0)';
          el.addEventListener('transitionend', done);
      })
    },
    afterDrop(el) {
      let ball = this.dropBalls.shift();
      if(ball) {
        ball.show = false;
        el.style.display = 'none';
      }
    }
  },
  components: {
    cartcontrol
  }
}
</script>
<style lang="stylus">
@import "../../common/stylus/mixin.styl"
  .shopcart
    position fixed
    left 0
    bottom 0
    z-index 50
    width 100%
    height 48px
    background red
    .content
      display flex
      background #141d27
      font-size 0
      color: rgba(255, 255, 255, 0.4)
      .content-left
        flex 1
        .logo-wrapper
          display inline-block
          vertical-align: top
          position relative
          top -10px
          margin 0 12px
          padding 6px
          width 56px
          height 56px
          box-sizing border-box
          border-radius 50%
          background #141d27
          .logo
            width 100%
            height 100%
            text-align center
            border-radius 50%
            background #2b343c
            &.highlight
              background: rgb(0, 160, 220)
            .icon-shopping_cart
              line-height 44px
              font-size 24px
              color #80858a
              &.highlight
                color: #fff
          .num
            position absolute
            top 0
            right 0
            width 24px
            height 16px
            line-height 16px
            text-align center 
            border-radius 16px
            font-size 9px
            font-weight 700
            color #fff
            background rgb(240, 20, 20)
            box-shadow 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display inline-block
          vertical-align: top
          margin 12px 0 0 12px
          font-size 16px
          padding-right 12px
          line-height 24px
          box-sizing border-box
          border-right 1px solid rgba(255, 255, 255, 0.1)
          font-weight 700
          &.highlight
            color: #fff
        .desc
          display inline-block
          line-height 24px
          vertical-align top
          font-size 10px
          margin 12px 0 0 12px
      .content-right
        flex 0 0 105px
        width 105px
        .pay
          height 48px
          line-height 48px
          text-align center
          font-size 12px
          font-weight 700
          &.not-enough
            background #2b343c
          &.enough
            background #00b43c
            color #fff
    .ball-container
      .ball
        position fixed
        left 32px
        bottom 22px
        z-index 200
        transition all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
        .inner
          width 16px
          height 16px
          border-radius 50%
          background rgb(0, 160, 220)
          transition all 0.4s linear
    .shopcart-list
      position absolute
      left 0
      top 0
      z-index -1
      width 100%
      transform translate3d(0, -100%, 0)
      &.fold-enter-active, &.fold-leave.active
        transition all 0.5s
      &.fols-enter, &.fold.leave-to
        transform translate3d(0, 0, 0)
      .list-header
        height 40px
        line-height 40px
        padding 0 18px
        background #f3f5f7
        border-bottom 1px solid rgba(7, 17, 27, 0.1)
        .title
          float left
          font-size 14px
          color rgb(7, 17, 27)
        .empty
          float right
          font-size 12px
          color rgb(0, 160, 220)
      .list-content
        padding 0 18px
        max-height 217px
        overflow hidden
        background #fff
        .food
          position relative
          padding 12px 0
          box-sizing border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            line-height 24px
            font-size 14px
            color rgb(7, 17, 27)
          .price
            position: absolute
            right: 90px
            bottom: 12px
            line-height: 24px
            font-size: 14px
            font-weight: 700
            color: rgb(240, 20, 20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px
  .list-mask
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    z-index: 40
    backdrop-filter: blur(10px)
    opacity: 1
    background: rgba(7, 17, 27, 0.6)
    &.fade-enter-active, &.fade-leave-active
      transition: all 0.5s
    &.fade-enter, &.fade-leave-active
      opacity: 0
      background: rgba(7, 17, 27, 0)        
</style>
