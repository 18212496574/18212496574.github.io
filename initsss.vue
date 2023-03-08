<template>
  <div>
    <avue-form v-model="formData" :option="form_option"  @submit="handleSubmit"></avue-form>

    <avue-crud ref="avueTable" :data="showData" :option="tableOption" :search.sync="search"
               :span-method="spanMethod"
    >
    </avue-crud>
  </div>
</template>

<script>

import NumFormat from "@/utils/NumFormat"
import AvueOption from "@/utils/AvueOption";
import UrlParams from "@/utils/UrlParams";
import ProductBillingService from "@/api/product_billing";
import LoadingFullscreen from "@/utils/LoadingFullscreen";
import AppTableFormat from "./DataConversion/AppTable"

export default {
  name: "ApplicationBills",
  data() {
    return {
      // 表单
      formData: {
        datetimerange: [UrlParams.getParam(this, "startTime"),UrlParams.getParam(this, "endTime")],
        appNames: JSON.parse(UrlParams.getParam(this, "appNames") === undefined ? "[]" : UrlParams.getParam(this, "appNames"))
      },
      form_option: {
        labelWidth:110,
        submitText: "搜索",
        column: [
          {
            label: "查询范围",
            type: 'datetimerange',
            prop:'datetimerange',
            span:12,
            defaultTime:['00:00:00', '23:59:59'],
            format:'yyyy-MM-dd HH:mm:ss',
            valueFormat:'yyyy-MM-dd HH:mm:ss',
            startPlaceholder: '时间日期开始范围自定义',
            endPlaceholder: '时间日期结束范围自定义',
            rules: [{
              required: true,
              message: "日期不可为空",
              trigger: "blur"
            }]
          },
          {
            label: '应用',
            prop: 'appNames',
            type: 'select',
            multiple: true,
            props: {
              label: 'name',
              value: 'code'
            },
            filterable: true,
            dicData: this.getAppList(),
            rules: [
              {
                required: true,
                message: '请选择应用',
                trigger: 'blur'
              }
            ]
          }
        ]
      },
      // 操作数据
      search: {
        ...UrlParams.getParamData(this)
      },
      showData: [],
      tableOption: {
        ...AvueOption.defaultBaseOption(),
        searchIndex: 3,
        column: [
          {
            label: '应用',
            prop: 'appName'
          },
          {
            label: '产品',
            prop: 'productDescription',
          },
          {
            label: '收费项',
            prop: 'chargeName',
            bind:'charge.name'
          },
          {
            label: '费用',
            prop: 'chargeAmount',
            bind:'charge.amount',
            formatter:(val,value,label)=>{
              // alert(JSON.stringify(val))
              return NumFormat.formatMoney(val['charge']['amount'])
            }
          },
          {
            label: '单价',
            prop: 'chargePrice',
            bind:'charge.price'
          },
          {
            label: '单价单位',
            prop: 'chargeUnit',
            bind:'charge.unit',
          },
          {
            label: '用量',
            prop: 'chargeVolume',
            bind:'charge.volume',
          },
          {
            label: '用量描述',
            prop: 'chargeVolumeDescription',
            bind:'charge.volumeDescription',
          }
        ]
      },
      spanArr: [],
      // 产品列表
      products: []
    }
  },
  created(){

  },
  mounted() {
    this.handleSubmit(this.formData , () => {})
  },
  methods: {
    // 合并列
    spanMethod({ row, column, rowIndex, columnIndex }) {
      for (let i = 0; i < this.spanArr.length; i++) {
        const ele = this.spanArr[i]
        if (column.property == ele.prop) {
          const _row = ele.span[rowIndex];
          const _col = _row > 0 ? 1 : 0;
          return {
            rowspan: _row,
            colspan: _col
          }
        }
      }
    },
    // 查询表单提交
    handleSubmit(form,done) {
      let queryForm = {
        "endTime": form['datetimerange'][1],
        "startTime": form['datetimerange'][0],
        "appNames": form['appNames'],
      }
      UrlParams.setParam(this, "endTime", queryForm.endTime)
      UrlParams.setParam(this, "startTime", queryForm.startTime)
      UrlParams.setParam(this, "appNames", JSON.stringify(queryForm.appNames))
      this.loadData(queryForm)
      done()
    },

    // 远程加载数据
    loadData(from) {
      this.products = this.$store.state.dict.products
      LoadingFullscreen.show(this)
      ProductBillingService.getAppBilling(from).then(res => {
        const {data, keysList} = AppTableFormat.conversion(res.result);
        this.showData = data
        this.spanArr = keysList
        console.log(res)
      }).finally(fin => {LoadingFullscreen.hide()})
    },

    getAppList() {
      let appNames = this.$store.state.dict.appNames
      let list = []
      for (let i = 0; i < appNames.length; i++) {
        list.push({name: appNames[i], code: appNames[i]})
      }
      return list
    }

  }
}
</script>
