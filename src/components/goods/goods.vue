/* eslint-disable no-unused-vars */
<template>
  <div class="goods" >
  <div class="menu-wrapper" ref="menuWrapper">
    <ul>
      <li v-for="(item,index) in goods.goods" :key="index" class="menu-item"
      :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
        <span class="text" >
          <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
          {{item.name}}
        </span>
      </li>
    </ul>
  </div>
  <div class="foods-wrapper" ref="foodsWrapper">
    <ul>
      <li v-for="(item,index) in goods.goods" :key="index" class="foods-list food-list-hook">
        <h1 class="title">{{ item.name }}</h1>
        <ul>
          <li  @click="selectFood(food,$event)" v-for="(food,i) in item.foods" :key="i" class="food-item">
            <div class="icon">
              <img :src="food.icon" width="57px" height="57px">
            </div>
            <div class="content">
              <h2 class="name">{{ food.name }}</h2>
              <p class="desc">{{ food.description }}</p>
              <div class="extra">
                <span class="count">月售{{ food.sellCount }}份</span>
                <span>好评率{{ food.rating }}%</span>
              </div>
              <div class="price">
                <span class="now">￥{{ food.price }}</span>
                <span class="old" v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
              </div>
              <div class="cartcontrol-Wrapper">
                <cartcontrol :food="food" @MyEvent="childEventListener"></cartcontrol>
              </div>
            </div>

          </li>
        </ul>
      </li>
    </ul>
  </div>
  <!-- 放入购物车-->
  <shopcart  :select-foods="selectFoods" :min-price="this.seller.minPrice" :delivery-price="this.seller.deliveryPrice"  :food-count="this.foodCount"></shopcart>
  <food @add="addFood" :food="selectedFood" ref="food"></food>
  </div>
</template>

<script>
import food from '../food/food'
import BScroll from '@better-scroll/core'
import axios from 'axios'
import cartcontrol from '../cartconted/cartcontrol'
import shopcart from '../shopcart/shopcart'
export default {
  name: 'goods',
  data () {
    return {
      goods: {
        goods: []
      },
      listHeight: [],
      scrollY: 0,
      selectedFood: {},
      foodCount: 0
    }
  },
  props: {
    seller: {
      type: Object
    }
  },
  computed: {
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        const height1 = this.listHeight[i]
        const height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i
        }
      }
      return 0
    },
    selectFoods() {
      const foods = []
      if (this.goods) {
        this.goods.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
      }
      return foods
    }
  },

  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
    axios.get('/getGoods').then(res => {
      const { data } = res
      this.goods = Object.assign({}, data)
      this.$nextTick(() => {
        this._initScroll()
        this._calculateHeight()
      })
    }).catch(err => {
      console.log(err)
    })
  },
  methods: {
    _initScroll() {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        probeType: 3,
        click: true
      })
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
      })
    },
    _calculateHeight() {
      const foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodList.length; i++) {
        const item = foodList[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    },
    selectMenu(index, event) {
      if (!event._constructed) {
        return
      }
      const foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      const el = foodList[index]
      this.foodsScroll.scrollToElement(el, 300)
    },
    selectFood(food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.food.show()
    },
    addFood(target) {
      this._drop(target)
    },
    childEventListener(data) {
      this.foodCount = data
      console.log(this.foodCount)
    }
  },
  components: {
    shopcart,
    cartcontrol,
    food
  }

}
</script>

<style lang="stylus" scoped>
 @import '../../mixins/Mixin.styl'
  .goods
    display flex
    position absolute
    width 100%
    top 174px
    bottom 46px
    overflow hidden
    .menu-wrapper
      flex 0 0 80px
      width 80%
      background #f3f5f7
      .menu-item
        display table
        height 50px
        line-height 14px
        &.current
          position relative
          z-index 10
          font-weight 700
          margin-top -1px
          background #fff
          color rgb(250,140,20)
          .text
            border-none()
        .text
          font-weight bolder
          text-align center
          display table-cell
          width 80px
          vertical-align middle
          font-size 12px
          border-1px (rgba(7,17,27,0.1))
        .icon
          display inline-block
          width 12px
          height 12px
          vertical-align top
          margin-right 2px
          background-size 12px 12px
          background-repeat no-repeat
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
    .foods-wrapper
      flex 1
      .title
        padding-left 14px
        height 26px
        line-height 26px
        border-left 2px solid #d9dde1
        font-size 12px
        color rgb(147,153,159)
        background #f3f5f7
      .food-item
        display flex
        margin 18px
        padding-bottom 18px
        border-1px (rgba(7,17,27,0.1))
        &:last-child
          border-none()
          margin-bottom 0
        .icon
          flex 0 0 57px
          margin-right 10px
        .content
          flex 1
          .name
            margin 2px 0 8px 0
            height 14px
            line-height 14px
            font-size 14px
            color rgb(7,17,27)
          .desc, .extra
            line-height 10px
            font-size 10px
            color rgb(147,153,159)
          .desc
            line-height 12px
             margin-bottom 8px
          .extra
            &.count
              margin-right 12px
          .price
            font-weight 700
            line-height 24px
            .now
              margin-right 8px
              font-size 14px
              color rgb(240,20,20)
            .old
              text-decoration line-through
              font-size 10px
              color rgb(147,153,159)
          .cartcontrol-Wrapper
            position absolute
            right 0
            bottom 12px
</style>
