<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评价</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <keep-alive>
      <router-view :seller="seller"/>
    </keep-alive> 
  </div>
</template>

<script>
import { urlParse } from 'common/js/util';
import header from '@/components/header/header.vue'
import axios from 'axios'
const ERR_OK = 0
export default {
  name: 'App',
  data(){
    return {
      seller: {
        id: (() => {
            let queryParam = urlParse();
            // console.log(queryParam);
            return queryParam.id;
        })()
      }
    }
  },
  created() {
      // 获取header组件的数据
      axios.get('api/seller' + '?id=' + this.seller.id).then((response) => {
        response = response.data;
        // console.log(response);
        if(response.errno === ERR_OK){
          // this.seller = response.data;
          this.seller = Object.assign({}, this.seller, response.data);
          // console.log(this.seller)
          // console.log(this.seller.avatar)
        }
        // console.log(response)
      })
  },
  components: {
   "v-header": header,
  }
}
</script>

<style lang="stylus">
  @import './common/stylus/mixin.styl'

  #app
    .tab
      display flex
      width 100%
      height 40px
      line-height 40px
      border-1px(rgba(7,17,27,0.1))
      .tab-item
        flex 1
        text-align center
        & > a
          display block 
          font-size 14px
          color rgb(77, 85, 93)
          &.active
            color: rgb(240,20, 20)
</style>

