<template>
  <div class="ratings" ref="ratings">
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <ratingselect  @select="selectRating" @toggle="toggleContent" :selectType="selectType" :onlyContent="onlyContent" :ratings="ratings"></ratingselect>
      <div class="rating-wrapper">
        <ul>
          <li v-for="item in ratings" :key="item.index" v-show="needShow(item.rateType, item.text)" class="rating-item">
            <div class="avatar">
              <img :src="item.avatar" width="28" height="28" alt="">
            </div>
            <div class="content">
              <h1 class="name">{{item.username}}</h1>
              <div class="star-wrapper">
                <star :size="24" :score="item.score"></star>
                <span class="delivery" v-show="item.deliveryTime">{{item.deliveryTime}}分钟</span>
              </div>
              <p class="text">{{item.text}}</p>
              <div class="recommend" v-show="item.recommend && item.recommend.length">
                <span class="icon-thumb_up"></span>
                <span class="item" v-for="item in item.recommend" :key="item.index">{{item}}</span>
              </div>
              <div class="time">
                {{item.rateTime | formatDate}}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script>
import BScroll from 'better-scroll';
import axios from 'axios'
import {formatDate} from 'common/js/date';
import star from '@/components/star/star.vue';
import split from '@/components/split/split.vue';
import ratingselect from '@/components/ratingselect/ratingselect.vue';
const ALL = 2;
const ERR_OK = 0;
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
      return {
        ratings: [],
        selectType: ALL,
        onlyContent: true,
      };
  },
  created() {
     // 获取ratings组件的数据
    axios.get('api/ratings').then((response) => {
      response = response.data;
      // console.log(response);
      if(response.errno === ERR_OK){
        this.ratings = response.data;
        // 数据更新及dom更新是异步的，如需要操作dom需要dom更新完成
        this.$nextTick(() => {
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            });
          })
        })
      }
    })
  },
  methods: {
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
    star,
    split,
    ratingselect
  }
}
</script>
<style lang="stylus">
@import "../../common/stylus/mixin.styl"
  .ratings
    position absolute
    top 174px
    bottom 0
    left 0
    width 100%
    overflow hidden
    .overview
      display flex
      padding 18px 0
      .overview-left
        flex 0 0 137px
        width 137px
        padding 6px 0
        text-align center
        border-right 1px solid rgba(7, 17, 27, 0.1)
        @media only screen and (max-width: 320px)
          flex: 0 0 120px
          width: 120px
        .score
          margin-bottom 6px
          line-height 28px
          font-size 24px
          color rgb(255, 153, 0)
        .title
          margin-bottom 8px
          font-size 12px
          line-height 12px
          color rgb(7, 17, 27)
        .rank
          line-height 10px
          font-size 10px
          color rgb(147, 153, 159)
      .overview-right
        flex 1
        padding: 6px 0 6px 24px
        @media only screen and (max-width: 320px)
          padding-left: 6px
        .score-wrapper
          margin-bottom 8px
          font-size 0
          .title
            display inline-block
            line-height 18px
            font-size 12px
            vertical-align: top
            color rgb(7, 17, 27)
          .star
            display inline-block
            margin 0 12px
            vertical-align: top
          .score
            display inline-block
            line-height 18px
            vertical-align: top
            font-size 12px
            color rgb(255, 153, 0)
        .delivery-wrapper
          font-size 0
          .title
            line-height 18px
            font-size 12px
            color rgb(7, 17, 27)
          .delivery
            margin-left 12px
            font-size 12px
            color rgb(147, 153, 159)
    .rating-wrapper
      padding 0 18px
      .rating-item
        display flex
        padding 18px 0
        border-1px(rgba(7, 17, 27, 0.1))
        .avatar
          flex 0 0 28px
          width 28px
          margin-right 12px
          img
            border-radius 50%
        .content
          flex 1
          position relative
          .name
            font-size 10px
            line-height 12px
            color rbg(7, 17, 27)
            margin-bottom 4px
          .star-wrapper
            margin-bottom 6px
            font-size 0
            .star
              display inline-block
              vertical-align: top
              margin-right 6px
            .delivery
              display inline-block
              vertical-align: top
              line-height 12px
              font-size 10px
              color rgb(147, 153, 159)
          .text
            margin-bottom 8px
            line-height 18px
            color rgb(7, 17, 27)
            font-size 12px
          .recommend
            line-height 16px
            font-size 0
            .icon-thumb_up, .item
              display inline-block
              margin 0 8px 4px 0
              font-size 9px
            .icon-thumb_up
              color: rgb(0, 160, 220)
            .item
              padding 0 6px
              border 1px solid rgba(7, 17, 27, 0.1)
              border-radius 1px
              color rgb(147, 153, 159)
              background #fff
          .time
            position: absolute
            top: 0
            right: 0
            line-height: 12px
            font-size: 10px
            color: rgb(147, 153, 159)

</style>
