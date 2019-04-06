<template>
  <div>
    <inputSelect
      ref="dealer_sel"
      :requestlist="dealerlist.requestlist"
      :placeholder="dealerlist.placeholder"
      :detalutname="dealerlist.name"
      @returnedvalue="selctderequestmall"
      @requestname="requestnamemall"
      @losebule="losebule"
    />
  </div>
</template>
<script>
import inputSelect from '~/components/input-select'
export default {
  data () {
    return {
      filter: {
        dealer_id: ''
      },
      searchLoading: false,
      dealerlist: {
        requestlist: [],
        placeholder: '请输入或查找门店',
        name: ''
      }
    }
  },
  components: {
    inputSelect
  },
  methods: {
    // 门店相关
    selctderequestmall (item) {
      this.filter.dealer_id = item.id
      this.dealerlist.requestlist = []
    },
    requestnamemall (_name) {
      this.filter.dealer_id = ''
      if (_name === '') {
        this.dealerlist.requestlist = []
      } else {
        this.$mallanddealer.dealerlist({ name: _name })
          .then(({ data }) => {
            this.dealerlist.requestlist = data.data.stores.length > 0 ? data.data.stores.map((item) => {
              return {
                address: item.address,
                area_code: item.area_code,
                area_id: item.area_id,
                city: item.city,
                created_at: item.created_at,
                full_name: item.full_name,
                geohash: item.geohash,
                id: item.id,
                is_support_sign_in: item.is_support_sign_in,
                mall: item.mall,
                mall_id: item.mall_id,
                name: `${item.mall}--${item.name}`,
                position: item.position,
                province: item.province,
                updated_at: item.updated_at
              }
            }) : [{ name: '无匹配数据' }]
          })
      }
    },
    losebule () {
      setTimeout(() => {
        if (this.filter.dealer_id === '') {
          this.dealerlist.requestlist = []
          this.$refs.dealer_sel.removename()
        }
      }, 400)
    }
  }
}
</script>
