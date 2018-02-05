<template>
  <div class="shopcart">
    <div class="content">
      <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo">
              <i class="icon-shopping_cart"></i>
            </div>
          </div>
          <div class="price">￥0</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
          <div class="pay">￥{{minPrice}}起送</div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  }
}
</script>
<style lang="stylus">
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
            .icon-shopping_cart
              line-height 44px
              font-size 24px
              color #80858a
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
        .desc
          display inline-block
          line-height 24px
          vertical-align top
          font-size 10px
          margin 12px 0 0 12px
      .content-right
        flex 0 0 105px
        width 105px
        background #2b343c
        .pay
          height 48px
          line-height 48px
          text-align center
          font-size 12px
          font-weight 700
</style>
