<template>
  <transition name="move">
    <div class="food" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="image-header">
            <!-- 图片加载是异步过程 -->
          <img :src="food.image" alt="">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food" @add="addFood"></cartcontrol>
          </div>
          <transition name="fade">
            <div @click.stop.prevent="addFirst" class="buy" v-show="!food.count || food.count===0">加入购物车</div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect @select="selectRating" @toggle="toggleContent" :ratings="food.ratings" :selectType="selectType" :onlyContent="onlyContent" :desc="desc"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-for="item in food.ratings" :key="item.index" v-show="needShow(item.rateType, item.text)" class="rating-item">
                <div class="user">
                  <span class="name">{{item.username}}</span>
                  <img :src="item.avatar" width="12" height="12" alt="">
                </div>
                <div class="time">{{item.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':item.rateType===0,'icon-thumb_down':item.rateType===1}"></span>{{item.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>
<script>
import BScroll from 'better-scroll';
import Vue from 'vue';
import {formatDate} from 'common/js/date';
import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
import split from '@/components/split/split.vue';
import ratingselect from '@/components/ratingselect/ratingselect.vue';
const ALL = 2;
export default {
  props: {
    food: {
      type: Object
    }
  },
  data() {
    return {
      showFlag: false,
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    }
  },
  methods: {
    //   显示商品详情页
    show() {
      this.showFlag = true;
      this.selectType = ALL;
      this.onlyContent = true;
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.food, {
            click: true
          });
        } else {
          // 重新计算 better-scroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常
          this.scroll.refresh();
        }
      });
    },
    // 返回到商品页
    hide() {
      this.showFlag = false;
    },
    addFirst(event) {
      Vue.set(this.food, 'count', 1);
      this.$emit('add', event.target);
    },
    addFood(target) {
      this.$emit('add', target);
    },
    selectRating(type) {
      this.selectType = type;
      this.$nextTick(() => {
          this.scroll.refresh();
      });
    },
    toggleContent() {
      this.onlyContent = !this.onlyContent;
      this.$nextTick(() => {
          this.scroll.refresh();
      });
    },
    needShow(type, text){
      // 打勾按钮显绿色 && 某个评价内容为空 => false 该评价所有信息不显示
      if(this.onlyContent && !text) {
        return false;
      }
      if(this.selectType === ALL) {//显示全部的评价信息
        return true;
      }else {// 只显示相应rateType值得评价信息
        // console.log(type === this.selectType);
        return type === this.selectType;
      }
    }
  },
  filters: {
    formatDate(time) {
      let date = new Date(time);
      return formatDate(date, 'yyyy-MM-dd hh:mm');
    }
  },
  components: {
    cartcontrol,
    split,
    ratingselect
  }
}
</script>
<style lang="stylus">
@import "../../common/stylus/mixin.styl"
  .food
    position fixed
    left 0
    top 0
    bottom 48px
    z-index 30
    width 100%
    background #fff
    overflow hidden
    opacity 1
    transform translate3d(0, 0, 0)
    &.move-enter-active, &.move-leave-active
      transition all 0.2s linear 
    &.move-enter, &.move-leave-to
      opacity 0
      transform translate3d(100%, 0, 0)
    .image-header
      position relative
      width 100%
      height 0
      padding-top 100%
      img
        position absolute
        top 0
        left 0
        width 100%
        height 100%
      .back
        position absolute
        top 10px
        left 10px
        .icon-arrow_left
          display block
          padding 10px
          font-size 20px
          color #fff
    .content
      position relative
      padding 18px
      .title
        line-height 14px
        margin-bottom 8px
        font-size 14px
        font-weight 700
        color rgb(7, 17, 27)
      .detail
        margin-bottom 18px
        line-height 10px
        height 10px
        font-size 0
        .sell-count, .rating
          font-size 10px
          color rgb(147, 153, 159)
        .sell-count
          margin-right 12px
      .price
        font-weight: 700
        line-height: 24px
        .now
          margin-right: 8px
          font-size: 14px
          color: rgb(240, 20, 20)
        .old
          text-decoration: line-through
          font-size: 10px
          color: rgb(147, 153, 159)
      .cartcontrol-wrapper
        position absolute
        right 18px
        bottom 18px
      .buy
        position absolute
        right 18px
        bottom 18px
        z-index 10
        height 24px
        line-height 24px
        padding 0 12px
        box-sizing border-box
        border-radius 12px
        font-size 10px
        color #fff
        background rgb(0, 160, 220)
        opacity 1
        &.fade-enter-active, &.fade-leave-active
         transition all 0.2s
        &.fade-enter, &.fade-leave-to
          opacity 0
          z-index -1
    .info
      padding 18px
      .title
        line-height 14px
        margin-bottom 6px
        font-size 14px
        color rgb(7, 17, 27)
      .text
        line-height 24px
        padding 0 8px
        font-size 12px
        color rgb(77, 85, 93)
    .rating
      padding-top 18px
      .title
        margin-left 18px
        line-height 14px
        font-size 14px
        color rgb(7, 17, 27)
      .rating-wrapper
        padding 0 18px
        .rating-item
          position relative
          padding 16px 0
          border-1px(rgba(7, 17, 27, 0.1))
          .user
            position absolute
            right 0
            top 16px
            line-height 12px
            font-size 0
            .name
              display inline-block
              font-size 10px
              margin-right 6px
              vertical-align top 
              color: rgb(147, 153, 159)
            .avatar
              border-radius 50%
          .time
            margin-bottom 6px
            line-height 12px
            font-size 10px
            color rgb(147, 153, 159)
          .text
            line-height 16px
            font-size 12px
            color rgb(7, 17, 27)
            .icon-thumb_up, .icon-thumb_down
              margin-right 4px
              line-height 16px
              font-size 12px
            .icon-thumb_up
              color rgb(0, 160, 220)
            .icon-thumb_down
              color rgb(147, 153, 159)
      
        .no-rating
          padding 16px 0
          font-size 12px
          color rgb(147, 153, 159)
</style>
