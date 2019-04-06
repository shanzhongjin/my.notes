
<template>
  <div class="detault_div">
    <el-input v-model.trim="name" :placeholder="placeholder" clearable  @input="cityfindfun" @focus.stop="clicksratio" style="width:100%;"></el-input>
    <!-- 蒙板 -->
    <div class="detault_shade" v-show="detault_shade" @click.stop="clickshade"></div>
    <div class="detault" v-show="is_default || requestlist.length > 0">
      <!-- 默认数据 -->
      <ul class="detault_ul" v-show="is_default">
        <li v-for="(item, index) in deafaultlist" :key="index" @click.stop="mallselctdefault(item)">{{item.name}}</li>
      </ul>
       <!-- 搜索的数据 -->
      <ul class="detault_ul" v-show="requestlist.length > 0">
        <li v-for="(item, index) in requestlist" :key="index" @click.stop="mallselctdefault(item)">{{item.name}}</li>
      </ul>
    </div>
  </div>
</template>
<script>
/**
 *  1. 点击 input 显示默认选项框
 *  3. input 输入值即时搜索数据并返回展示
 *  4. 点击 搜索的数据 搜索下拉框隐藏
 *  5. 点击蒙板 下拉框隐藏
 *  6. 清除input的值的时候默认的数据显示
 *  遮罩层 z-index为5
 */
export default {
  name: 'malldealer',
  props: {
    placeholder: {
      type: String,
      default: ''
    },
    requestlist: {
      type: Array,
      default: () => []
    },
    deafaultlist: {
      type: Array,
      default: () => []
    },
    detalutname: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      name: '',
      is_default: false,
      detault_shade: false
    }
  },
  watch: {
    // name: {}
  },
  // computed () {},
  created () {
    this.name = this.detalutname
  },
  mounted () {
  },
  methods: {
    // 点击蒙板
    clickshade () {
      this.detault_shade = false
      this.is_default = false
      this.$emit('closelist')
    },
    // input聚焦
    clicksratio () {
      const that = this
      that.detault_shade = true
      that.is_default = true
      that.$emit('closelist')
    },
    // 选中数据
    mallselctdefault (item) {
      this.$emit('closelist')
      this.$emit('returnedvalue', item.id)
      this.name = item.name
      this.is_default = false
      this.detault_shade = false
    },
    // 输入值变动
    cityfindfun (val) {
      if (val === '') {
        this.$emit('requestname', '')
        this.is_default = true
        this.detault_shade = true
      } else {
        this.is_default = false
        this.$emit('requestname', val)
      }
    },
    // 清空输入值（父级调用）
    removename () { this.name = '' }
  }
}
</script>
<style lang="scss" scoped>
.detault{
  position: relative;
  width: 100%;
  height: 210px;
  padding-left: 2px;
  overflow: hidden;
  &_shade{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 5;
    background-color: 000;
    opacity: 0.3;
  }
  &_div{
    width: 100%;
    height: 40px;
  }
  &_ul{
    position: absolute;
    top: 5px;
    left: 0;
    z-index: 9;
    padding:3px 5px 3px;
    width: 100%;
    margin-right: 17px;
    max-height: 190px;
    overflow-x: hidden;
    border-radius: 3px;
    background-color: #fff;
    box-shadow: 0 1px 6px rgba(0,0,0,.2);
    > li{
      width: 100%;
      box-sizing: border-box;
      line-height: normal;
      padding: 7px 16px;
      clear: both;
      color: #495060;
      font-size: 12px!important;
      white-space: nowrap;
      list-style: none;
      cursor: pointer;
      transition: background .2s ease-in-out;
      &:hover{
        background-color: #f3f3f3;
      }
    }
  }
}
</style>
