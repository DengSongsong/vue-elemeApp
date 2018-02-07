<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-decrease" @click="decreaseCart" v-show="food.count>0">
          <span class="inner icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click="addCart"></div>
  </div>
</template>
<script>
import Vue from 'vue'
export default {
  props: {
      food: {
          type: Object
      }
  },
  methods: {
    //   添加商品
    addCart(event) {
      if(!this.food.count){
        Vue.set(this.food, 'count', 1);
      }else { 
        this.food.count++;
      }
      // console.log(event.target);
      //向父组件传递子组件参数(数据)，本数据就是点击增加商品数量的那个
      // 按钮元素(目的在于知道是哪个商品(哪个按钮被触发了)要添加数量实现购物小球下落的效果)
      this.$emit('add',event.target);
    },
    // 减少商品数量
    decreaseCart(){
      if (this.food.count) {
        this.food.count--;
      }
    }
  }
}
</script>
<style lang="stylus">
  .cartcontrol
    font-size 0
    .cart-decrease
      display inline-block
      vertical-align: top
      opacity 1
      transform translate3d(0, 0, 0)
      .inner
        display inline-block
        font-size 24px
        line-height 24px
        color rgb(0, 160, 220)
        transition all 0.4s linear 
        transform rotate(0)
      &.move-enter-active, &.move-leave-active
        transition all 0.4s linear 
      &.move-enter, &.move-leave-to
        opacity 0
        transform translate3d(24px, 0, 0)
        .inner
          transform rotate(180deg)
    .cart-count
      display inline-block
      vertical-align: top
      width 12px
      padding-top: 6px
      font-size 10px
      text-align center 
      color rgb(147, 153, 159)
    .cart-add
      display inline-block
      vertical-align: top
      font-size 24px
      line-height 24px
      color rgb(0, 160, 220)
</style>
