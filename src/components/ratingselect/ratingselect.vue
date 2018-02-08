<template>
  <div class="ratingselect">
    <div class="rating-type">
      <span class="block positive" @click="select(2)" :class="{'active':selectType===2}">{{desc.all}}<span class="count">{{ratings.length}}</span></span>
      <span class="block positive" @click="select(0)" :class="{'active':selectType===0}">{{desc.positive}}<span class="count">{{positives.length}}</span></span>
      <span class="block negative" @click="select(1)" :class="{'active':selectType===1}">{{desc.negative}}<span class="count">{{negatives.length}}</span></span>
    </div>
    <div class="switch" :class="{'on':onlyContent}" @click="toggleContent">
      <span class="icon-check_circle"></span>
      <span class="text">只看有内容的评价</span>
    </div>
  </div>
</template>
<script>
const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;
export default {
  props: {
    ratings: {
      type: Array,
      default() {
          return [];
      }
    },
    selectType: {
      type: Number,
      default: ALL
    },
    onlyContent: {
      type: Boolean,
      default: false
    },
    desc: {
      type: Object,
      default() {
        return {
          all: '全部',
          positive: '满意',
          negative: '不满意'
        };
      }
    }
  },
  computed: {
    //   rateType:0的评价
    positives() {
      return this.ratings.filter((rating) => {
        return rating.rateType === POSITIVE;
      });
    },
    //   rateType:1的评价
    negatives() {
      return this.ratings.filter((rating) => {
        return rating.rateType === NEGATIVE;
      });
    }
  },
  methods: {
    select(type) {
    //   console.log('click');
      this.$emit('select', type);
    },
    toggleContent() {
    //   console.log('click');
    //   this.onlyContent = !this.onlyContent;
      this.$emit('toggle');
    }
  }
}
</script>
<style lang="stylus">
  @import "../../common/stylus/mixin.styl"
  .ratingselect
    .rating-type
      padding 18px 0
      margin 0 18px
      font-size 0
      border-1px(rgba(7, 17, 27, 0.1))
      .block
        display inline-block
        padding 8px 12px
        margin-right 8px
        font-size 12px
        line-height 16px
        border-radius 1px
        color: rgb(77, 85, 93)
        &.positive
          background rgba(0, 160, 220, 0.2)
          &.active
            background: rgb(0, 160, 220)
            color #fff
          .count
            margin-left 2px
            font-size 8px
        &.negative
          background rgba(77, 85, 93, 0.2)
          &.active
            background: rgb(77, 85, 93)
            color #fff
    .switch
      padding 12px 18px 
      line-height 24px
      border-bottom 1px solid rgba(7, 17, 27, 0.1)
      color rgb(147, 153, 159)
      font-size 0
      &.on
        .icon-check_circle
          color: #00c850
      .icon-check_circle
        display inline-block
        vertical-align top
        margin-right 4px
        font-size 24px
      .text
        display inline-block
        vertical-align top
        font-size 12px
</style>
