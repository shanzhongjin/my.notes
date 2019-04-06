<template>
  <div>
    <input-select-defailt
      :deafaultlist="malllist.deafaultlist"
      :requestlist="malllist.requestlist"
      :placeholder="malllist.placeholder"
      :detalutname="form.mall_name"
      @returnedvalue="selctderequestmall"
      @requestname="requestnamemall"
      @closelist="closemalllist"
    />
  </div>
</template>
<script>
import inputSelectDefailt from '~/components/input-select-defailt'
export default {
  async asyncData ({ app, params }) {
    const form = {
      id: params.id,
      address: '',
      mall_id: '',
      mall_name: '',
    }
    const malls = await app.$mallanddealer.malllist({ limit: 15, page: 1 }) // 默认卖场列表
    const mallsarr = malls.data.data.malls.map((item) => {
      return {
        id: item.id,
        name: item.name
      }
    })
    return {
      malllist: {
        requestlist: [],
        deafaultlist: mallsarr,
        placeholder: '请输入或选择卖场'
      },
      form
    }
  },
  data () {
    return {}
  },
  components: {
    inputSelectDefailt
  },
  methods: {
    // 卖场
    selctderequestmall (_id) {
      this.form.mall_id = _id
    },
    // 根据 mall name request mallliist
    requestnamemall (_name) {
      if (_name === '') {
        this.form.mall_id = ''
      }
      this.$mallanddealer.malllist({ name: _name })
        .then(({ data }) => {
          this.malllist.requestlist = data.status === 'success' ? data.data.malls.map((item) => {
            return {
              id: item.id,
              name: item.name
            }
          }) : []
        })
    },
    closemalllist (_str) {
      this.malllist.requestlist = []
    }
  }
}
</script>
