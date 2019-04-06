
<template>
  <div class="detault_div">
    <el-input v-model.trim="name" :placeholder="placeholder" clearable  @input="cityfindfun" @blur="lose" style="width:100%;"></el-input>
    <div class="detault" v-show="requestlist.length > 0">
       <!-- 搜索的数据 -->
      <ul class="detault_ul" v-if="requestlist.length > 0">
        <li v-for="(item, index) in requestlist" :key="index" @click.stop="mallselctderequest(item)">{{item.name}}</li>
      </ul>
    </div>
  </div>
</template>
<script>
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
    detalutname: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      name: ''
    }
  },
  created () {
    this.name = this.detalutname
  },
  methods: {
    lose () {
      this.$emit('losebule')
    },
    mallselctderequest (item) {
      if (item.id) {
        this.name = item.name
        this.$emit('returnedvalue', item)
      }
    },
    cityfindfun (val) {
      this.$emit('requestname', val)
    },
    removename () {
      this.name = ''
    }
  }
}
</script>
<style lang="scss" scoped>
.detault{
  position: relative;
  width: 100%;
  padding-left: 2px;
  &_div{
    height: 40px;
  }
  &_ul{
    position: absolute;
    top: 5px;
    left: 0;
    z-index: 9;
    padding-right:3px 5px 3px;
    width: 100%;
    max-height: 190px;
    overflow-x: hidden;
    overflow-y: scroll;
    border-radius: 3px;
    background-color: #fff;
    box-shadow: 0 1px 6px rgba(0,0,0,.2);
    > li{
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
