<template>
  <div>
    <button
      class="buy"
      hover-class="none"
      :disabled="good.inventory <= 0"
      @click="popshowfun"
    >立即购买</button>
    <!-- 弹窗 -->
    <view class="popup" v-show="popShow" @click="popshowfun"></view>
    <view class="emption" v-show="popShow">
      <view class="emption-header">
        <view class="emption-header__cover">
          <image class="wh_100" :src="good.cover_url"></image>
        </view>
        <view class="emption-header__main">
          <view class="emption-header__name">{{good.title}}</view>
          <view class="emption-header__price" v-if="good.type === 'POINT'">
            <view v-show="curr_is_str !== ''">
              {{curr.point * shopNumber}}积分
              <text class="emption-header__point">{{curr.inventory}}库存</text>
            </view>
            <view v-show="curr_is_str === ''">
              {{ good.point * shopNumber}}积分 
              <text class="emption-header__point">{{good.inventory}}库存</text>
            </view>
          </view>
          <view class="emption-header__price" v-else>
            <text v-show="curr_is_str !== ''"> ￥{{curr.price * shopNumber}}</text>
            <text v-show="curr_is_str === ''"> ￥{{good.price * shopNumber}}</text>
            <text class="emption-header__join">+</text>
            <view v-show="curr_is_str !== ''">
              {{curr.point * shopNumber}}积分
              <text class="emption-header__point">{{curr.inventory}}库存</text>
            </view>
            <view v-show="curr_is_str === ''">
              {{good.point * shopNumber}}积分
              <text class="emption-header__point">{{good.inventory}}库存</text>
            </view>
          </view>
        </view>
        <view class="emption-header__img" @click="popshowfun">
          <image class="emption-header__close" src="/static/img/mybacode_close.png"></image>
        </view>
      </view>
      <block v-if="skuFliter.length > 0">
        <scroll-view scroll-y class="sku">
          <view class="sku-list" v-for="(item, index) in skuname" :key="item.id">
            <view class="sku-list__name">{{item.name}}</view>
            <view class="sku-list__option">
              <block v-for="(itemsub, indexsub) in item.term" :key="indexsub">
                <div
                  v-if="itemsub.status"
                  class="sku-list__tab"
                  :class="{'tab_curr': item.pitch === itemsub.value}"
                  @click="taptab(index, itemsub.value)"
                  >{{itemsub.value}}
                  </div>
                <!-- <div v-else class="sku-list__tab" >-{{itemsub.value}}</div> -->
              </block>
            </view>
          </view>
        </scroll-view>
      </block>
      <view class="emption-quantity">
        <text class="emption-quantity__name">数量</text>
        <view
          class="emption-quantity__btn"
          :class="{'disabled_btn': shopNumber <= 1}"
          @click="shopNumbersubtract"
        >-</view>
        <input class="emption-quantity__input" v-model="shopNumber"></input>
        <view
          class="emption-quantity__btn"
          :class="{'disabled_btn': !is_inventory}"
          @click="shopNumberadd"
        >+</view>
      </view>
        <button
          class="emption-btn"
          hover-class="none"
          :disabled="curr_is_str === ''"
          @click="confirmfun"
        >确定</button>
    </view>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // userIsExist: false, // 判断用户是否存在
      id: '',
      popShow: false, // 弹窗显示隐藏
      good: {
        banner_images: []
      },
      shopNumber: 1,
      skuall: {}, // 全部的(有库存并且启用的商品规格)
      skuname: [], // 可用的
      skuFliter: [], // sku的过滤版
      skutemp: [], // 记录二位数组方便赋值
      curr: {
        point: '',
        price: '',
        inventory: ''
      },
      pitch: '' // 选中的规格
    }
  },
  computed: {
    is_inventory () {
      const inventory = this.curr.inventory || this.good.inventory
      return this.shopNumber < Number(inventory)
    },
    curr_is_str () {
      return this.curr.point + this.curr.price + this.curr.inventory
    }
  },
  methods: {
    details (_id) {
      this.$http.goodsdetails(_id)
        .then((data) => {
          if (data.status === 'success') {
            this.good = data.data.good
            this.skuallfun()
          }
        })
    },
    skuallfun () {
      const skuname = []
      if (this.good.sku.length > 0) {
        this.good.sku[0].name.forEach((item, index) => {
          skuname[index] = {
            name: this.good.term[index].name,
            pitch: '',
            term: []
          }
          this.skutemp.push([])
        })
      }
      for (const item of this.good.sku) {
        if (item.inventory > 0 && item.is_use > 0) {
          this.skuFliter.push(item.name)
          const name = item.name.join()
          this.skuall[name] = {
            point: item.point,
            price: item.price,
            inventory: item.inventory
          }
          item.name.forEach((el, idl) => {
            skuname[idl].term.push(el)
          })
        }
      }
      skuname.forEach((item, index) => {
        const arr = Array.from(new Set(item.term))
        skuname[index].term = arr.map((item) => {
          return {
            value: item,
            status: true
          }
        })
      })
      this.skuname = skuname
    },
    // 其他规格未选择则 只展示能选择的选项
    taptab (_index, _itemsub) {
      if (this.skuname[_index].pitch === _itemsub) {
        this.skuname[_index].pitch = ''
      } else {
        this.skuname[_index].pitch = _itemsub
      }
      const arr = this.skuname.map((item) => { return item.pitch })
      // 判断是不是全空值
      if (/^\,*$/.test(arr.join(','))) { // eslint-disable-line
        this.currOperator('')
        this.skuname.forEach((item) => {
          item.term.forEach((iteml) => {
            iteml.status = true
          })
        })
      } else {
        if (arr.includes('')) {
          this.currOperator('')
          // 其他规格已选择则展示信息
          const abs = this.filSkuname(arr)
          this.skuname.forEach((item, index) => {
            if (item.pitch === '') {
              item.term.forEach((iteml, indexl) => {
                iteml.status = abs[index].includes(iteml.value)
              })
            }
          })
        } else {
          const pitch = arr.join()
          const curr = this.skuall[pitch]
          this.pitch = pitch
          this.currOperator(curr)
        }
      }
    },
    filSkuname (_arr) {
      const arr = _arr
      const skuFliter = this.skuFliter
      let fliearr = []
      for (let i = 0; i < arr.length; i++) {
        if (arr[i] !== '') {
          if (fliearr.length === 0) {
            fliearr = this.filSkuall(skuFliter, arr[i], i)
          } else {
            fliearr = this.filSkuall(fliearr, arr[i], i)
          }
        }
      }
      const arrtemp = JSON.parse(JSON.stringify(this.skutemp))
      fliearr.forEach((item, index) => {
        item.forEach((iteml, indexl) => {
          arrtemp[indexl].push(iteml)
        })
      })
      arrtemp.forEach((item, index) => {
        arrtemp[index] = Array.from(new Set(item))
      })
      return arrtemp
    },
    filSkuall (_arr, _el, _ind) {
      return _arr.filter((item) => {
        return item[_ind] === _el
      })
    },
    currOperator (_obj) {
      if (_obj === '') {
        this.curr.point = ''
        this.curr.price = ''
        this.curr.inventory = ''
      } else {
        this.curr.point = _obj.point
        this.curr.price = _obj.price
        this.curr.inventory = _obj.inventory
      }
    },
    confirmfun () {
      this.$router.push({path: '/pages/affirmorder/main',
        query: {
          point: this.curr.point,
          price: this.curr.price,
          inventory: this.curr.inventory,
          pitch: this.pitch,
          shopNumber: this.shopNumber,
          id: this.id
        }
      })
    },
    popshowfun () {
      this.popShow = !this.popShow
    },
    shopNumberadd () {
      if (this.is_inventory) {
        this.shopNumber += 1
      }
    },
    shopNumbersubtract () {
      if (this.shopNumber > 1) {
        this.shopNumber -= 1
      }
    }
  },
  onLoad (option) {
    const query = this.$route.query
    const id = query.id || option.id
    this.id = id
    this.details(id)
  },
  // 商品转发
  onShareAppMessage () {
    return {
      title: this.good.title,
      path: `/pages/index/main?id=${this.good.id}&type=goodsdetails`,
      imageUrl: this.good.cover_url // 拿第一张商品的图片
    }
  }
}
</script>
<style scoped lang="scss">
  .buy,
  .buy2{
    position: fixed;
    bottom: 0;
    left: 0; 
    z-index: 1;
    @include btn-fixed-bottom ($height: 104rpx,$backcolor: #e21f5b)
  }
  .buy2{
    z-index: 2;
  }
  .popup{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 4;
    width: 100%;
    height: 100vh;
    // background-color: $color-back-f5;
    background-color: transparent;
  }
  .emption{
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 5;
    width: 750rpx;
    background-color: $color-back-f;
    font-size: 28rpx;
    color: $color-text9;
    &-header{
      @include flex-x;
      padding: 30rpx;
      &__cover{
        width: 156rpx;
        height: 156rpx;
        border: 1px solid $border-f;
        background-color: $color-back-f5;
      }
      &__main{
        width: 428rpx;
        height: 160rpx;
        padding-left: 26rpx;
        @include flex-y;
        @include flex-space-b;
      }
      &__name{
        font-size: 30rpx;
        font-weight: 500;
        @include text_ellipsis2;
        color: #080404;
      }
      &__price{
        height: 32rpx;
        @include flex-middle;
        font-size: 28rpx;
        color: #e21f5b;
      }
      &__join{
        font-size: 22rpx;
        color: $color-text0;
      }
      &__img{
        flex-grow: 1;
        text-align: right;
      }
      &__close{
        width: 25rpx;
        height: 25rpx;
      }
      &__point{
        font-size: 24rpx;
        color: #666;
      }
    }
    &-quantity{
      height: 145rpx;
      padding:0 30rpx 25rpx;
      @include flex-middle;
      &__name{
        flex-grow: 1;
      }
      &__btn{
        width: 76rpx;
        height: 76rpx;
        border: 1px solid $border-f;
        @include flex-cen-cen;
        font-size: 40rpx;
        color: #000;
      }
      &__input{
        width: 114rpx;
        height: 76rpx;
        border-top: 1px solid $border-f;
        border-bottom: 1px solid $border-f;
        text-align: center;
        line-height: 76rpx;
      }
    }
    &-btn{
      @include btn-fixed-bottom ($height: 104rpx,$backcolor: #e21f5b)
    }
  }
  .sku{
    // height: 454rpx;
    height: 380rpx;
    border-top: 20rpx solid $color-back-f5;
    &-list{
      padding: 20rpx 0 30rpx 30rpx;
      border-bottom: 1px solid $border-f;
      &__name{
        height: 45rpx;
        @include flex-middle;
      }
      &__option{
        @include flex-wrap;
      }
      &__tab{
        margin-top: 20rpx;
        margin-right: 30rpx;
        line-height: 60rpx;
        min-width: 33rpx;
        height: 60rpx;
        padding: 0 22rpx;
        border: 2rpx solid transparent;
        background-color: $color-back-f5;
      }
    }
  }
  .tab_curr{
    border-color: #000;
    background-color: $color-back-f;
  }
  .disabled_btn{
    color: $color-disabled;
  }
  .dis-f{
    display: flex;
  }
  /**
    // @import '../../common/scss/mixin.scss';

  $color-back0b: #00b2ba; // 按钮背景色
  $color-back-f5: #f5f5f5; // 背景色
  $color-back-f: #fff; // 背景色
  $color-disabled: #999; // 禁止色
  $color-text9: #999; // 文本色
  $color-f: #fff; // 文本色
  $color-text0: #080404; // 文本色
  $highlight: #27bec5; // 高亮色
  $border-f: #f0f0f0; // 边框色

  // 不换行
  @mixin no-wrap {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  @mixin text_ellipsis2 {
    text-overflow: -o-ellipsis-lastline;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical; 
  }
  @mixin text_through {
    text-decoration: line-through;
  }
  @mixin flex {
    display: flex;
  }
  @mixin flex-x {
    display: flex;
    flex-direction: row;
  }
  @mixin flex-y {
    display: flex;
    flex-direction: column;
  }
  @mixin flex-alignend {
    display: flex;
    align-items: flex-end;
  }
  @mixin flex-middle {
    display: flex;
    align-items: center;
  }
  @mixin flex-wrap{
    @include flex;
    flex-wrap: wrap;
  }
  @mixin flex-space-b {
    justify-content: space-between;
  }
  @mixin flex-cen-cen {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  @mixin btn-fixed-bottom ($width: 100%, $height: 100%,$backcolor: $color-back0b){
    width: $width;
    height: $height;
    @include flex-cen-cen;
    font-size: 32rpx;
    background-color: $backcolor;
    color: $color-f;
  }
  */
</style>
