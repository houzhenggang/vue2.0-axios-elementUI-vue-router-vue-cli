<template>
  <div>
    <menu-left :routeIndex="menuLeftIndex"></menu-left>
    <div class="add-activity-object">
      <div class="add-activity-content">
        <div class="bread-bar plate flex">
          <div class="add-edit">
            拼团 > 新增
          </div>
          <el-button class="save"  type="success" @click="submitForm('ruleForm')" size="small">保存</el-button>
        </div>
        <div class="groupbuy-setting-list plate">
          <div class="title">拼团基础设置</div>
          <el-form :model="ruleForm" :rules="rules"  ref="ruleForm" label-width="170px">
            <el-form-item label="团长免单：">
              <el-radio-group v-model="ruleForm.exemption">
                <el-radio :label="1"> 免单 </el-radio>
                <el-radio :label="2"> 不免单 </el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="活动起止时间："  prop="rangeTime">
              <el-date-picker
                v-model="ruleForm.rangeTime"
                type="datetimerange"
                value-format="timestamp"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期">
              </el-date-picker>
            </el-form-item>
            <el-form-item label="拼团失效时间："  prop="outTime">
              <el-input v-model="ruleForm.outTime" maxlength="5" class="outTime" placeholder="拼团失效时间为1-23小时"></el-input>
            </el-form-item>
            <el-form-item label="开团人数：" prop="groupNum">
              <el-input v-model="ruleForm.groupNum"  class="groupNum"  maxlength="10" placeholder="开团人数至少为2人" ></el-input>
            </el-form-item>
            <el-form-item label="单个用户购买件数上限："  prop="limitNum">
              <el-input v-model="ruleForm.limitNum"  class="limitNum"  maxlength="10" placeholder="设置为0，表示不设上限"></el-input>
              <div class="tips">设置为0，表示不设上限</div>
            </el-form-item>
            <el-form-item>
            </el-form-item>
          </el-form>
        </div>
        <div class="groupbuy-setting-list plate" v-if="isGroupSet">
          <div class="title">批量数据修改</div>
          <el-form :model="setPrice" :rules="rule"  ref="setPrice" label-width="100px">
            <el-form-item label="拼团价格："  prop="groupPrice">
              <el-input v-model="setPrice.groupPrice"  class="groupPrice"></el-input>
            </el-form-item>
            <el-form-item label="库存："  prop="totalCount">
              <el-input v-model.number="setPrice.totalCount"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary"  @click="submitGroupForm('setPrice')">修改</el-button>
            </el-form-item>
          </el-form>
        </div>
        <div class="datalist">
          <div class="clear setGroup">
            <el-row>
              <el-button class="left"  @click="groupUpdate()" :disabled="multipleSelection.length == 0" size="small">批量修改</el-button>
              <el-button class="left"  @click="groupDelete()" :disabled="multipleSelection.length == 0" size="small">批量删除</el-button>
              <button class="right"  v-if="newCreat == 'newCreat'" @click="addGroupBuy()">新增商品</button>
              <button class="right"  v-if="newCreat != 'newCreat'" @click="addGroupSku()">新增规格</button>
            </el-row>
          </div>
          <el-table
            ref="multipleTable"
            :data="goods"
            tooltip-effect="dark"
            style="width: 100%"
            border
            @selection-change="handleSelectionChange">
            <el-table-column
              type="selection"
              label="全选"
              width="100">
            </el-table-column>
            <el-table-column
              label="商品"
              show-overflow-tooltip>
              <template slot-scope="scope">
                <div class="goods-info-box">
                  <span class="goods-img"><img :src="yiqixuanDomainUrl + scope.row.cover_url" alt=""></span>
                  <div class="goods-info">
                    <p class="goods-info-name">{{scope.row.goods_name}}</p>
                  </div>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="stock_count"
              label="规格"
              width="230"
              show-overflow-tooltip>
              <template slot-scope="scope">
                <span class="goods-info-price" v-if="scope.row.sku_desc">{{scope.row.sku_desc}}</span>
                <span class="goods-info-price" v-else>{{showSpecs(scope.row)}}</span>
              </template>
            </el-table-column>
            <el-table-column
              prop="price"
              label="原价"
              width="110"
              show-overflow-tooltip>
              <template slot-scope="scope">
                <span class="goods-info-price">￥{{ scope.row.price| money}}</span>
              </template>
            </el-table-column>
            <el-table-column
              label="拼团价格"
              width="110"
              show-overflow-tooltip>
              <template slot-scope="scope">
                <el-input    v-model="scope.row.prices"  @blur="setGroupMoney(scope.$index)"></el-input>
              </template>
            </el-table-column>
            <el-table-column
              prop="stock_count"
              label="现有库存"
              width="110"
              show-overflow-tooltip>
            </el-table-column>
            <el-table-column
              label="库存"
              width="110"
              show-overflow-tooltip>
              <template slot-scope="scope">
                <el-input   v-model="scope.row.stock_counts"  @blur="setGroupCount(scope.$index)"></el-input>
              </template>
            </el-table-column>
            <el-table-column
              label="操作"
              width="110">
              <template slot-scope="scope">
                <el-button @click="deleteSku(scope.$index)" type="text" class="delete-btn">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      <select-group-production  :yiqixuanDomainUrl="yiqixuanDomainUrl"   @goodsId="getGoodsId" @handleClose="getHandleClose" :goods-dialog-visible="goodsDialogVisible"></select-group-production>
    </div>
  </div>
</template>

<script>
import menuLeft from '@/components/menu-left'
import { mapState } from 'vuex'
import selectGroupProduction from '@/components/select-group-production'
import { groupGoodSku, setGroupInfo, getGroupInfo, updateGroupInfo } from '@/axios/api'
export default {
  data () {
    let validateRangeTime = (rule, value, callback) => {
      let time = new Date().getTime()
      if (value == '') {
        callback(new Error('活动起止时间不能为空'))
      } else if (value[1] - time < 24 * 60 * 60 * 1000) {
        callback(new Error('活动结束时间必须大于当前时间24小时'))
      } else {
        callback()
      }
    }
    let validateOutTime = (rule, value, callback) => {
      value = String(value)
      if (value == '') {
        callback(new Error('失效时间不能为空'))
      } else if (value > 23 || value < 1) {
        callback(new Error('失效时间大于1小时，小于23小时'))
      } else if (value.indexOf('.') > -1) {
        callback(new Error('失效时间必须是整数'))
      } else if (isNaN(value)) {
        callback(new Error('失效时间必须是数字'))
      } else {
        callback()
      }
    }
    let validateGroupNum = (rule, value, callback) => {
      value = String(value)
      if (value == '') {
        callback(new Error('开团人数不能为空'))
      } else if (value < 2) {
        callback(new Error('开团人数大于2人'))
      } else if (value.indexOf('.') > -1) {
        callback(new Error('开团人数必须是整数'))
      } else if (isNaN(value)) {
        callback(new Error('开团人数必须是数字'))
      } else {
        callback()
      }
    }
    let validateLimitNum = (rule, value, callback) => {
      value = String(value)
      if (value == '') {
        callback(new Error('单个用户购买件数上限不能为空'))
      } else if (isNaN(value)) {
        callback(new Error('单个用户购买件数必须是数字'))
      } else if (value.indexOf('.') > -1) {
        callback(new Error('单个用户购买件数必须整数'))
      } else if (value != 0 && value < 1) {
        callback(new Error('单个用户购买件数要大于1哦'))
      } else {
        callback()
      }
    }
    let validateGroupPrice = (rule, value, callback) => {
      value = value * 100
      var b = this.multipleSelection.every(function (v) {
        return value < v.price
      })
      if (value == '') {
        callback(new Error('批量修改价格不能为空'))
      } else if (isNaN(value)) {
        callback(new Error('批量修改价格必须是数字'))
      } else if (!b) {
        callback(new Error('批量修改价格必须比所选规格的原价低'))
      } else {
        callback()
      }
    }
    let validateTotalCount = (rule, value, callback) => {
      var b = this.multipleSelection.every(function (v) {
        return value <= v.total_stock_count
      })
      if (value == '') {
        callback(new Error('批量修改库存不能为空'))
      } else if (isNaN(value)) {
        callback(new Error('批量修改库存必须是数字'))
      } else if (!b) {
        callback(new Error('批量修改库存必须比所选规格的现有库存低'))
      } else {
        callback()
      }
    }
    return {
      ruleForm: {
        rangeTime: [],
        outTime: '',
        groupNum: '',
        limitNum: '3',
        exemption: 2,
        obj: {}
      },
      setPrice: {
        groupPrice: '',
        totalCount: ''
      },
      isGroupSet: false,
      activityTime: '',
      goodsImgSrc: '',
      routerIf: true,
      goodsId: undefined,
      goodsDialogVisible: false,
      good: [],
      newCreat: '',
      menuLeftIndex: '7-3',
      specialOffer: '',
      originalPrice: '',
      stock: '',
      goods: [],
      method: 'post',
      multipleSelection: '',
      productId: '',
      goodsList: [],
      // 拼团基础设置
      rules: {
        rangeTime: [
          { required: true, validator: validateRangeTime, trigger: ['blur', 'change'] }
        ],
        outTime: [
          { required: true, validator: validateOutTime, trigger: ['blur', 'change'] }
        ],
        groupNum: [
          { required: true, validator: validateGroupNum, trigger: ['blur', 'change'] }
        ],
        limitNum: [
          { required: true, validator: validateLimitNum, trigger: ['blur', 'change'] }
        ]
      },
      // 批量数据修改
      rule: {
        groupPrice: [
          { required: true, validator: validateGroupPrice, trigger: ['blur', 'change'] }
        ],
        totalCount: [
          { required: true, validator: validateTotalCount, trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  computed: {
    ...mapState(['menuLeft', 'yiqixuanDomainUrl'])
  },
  components: {
    menuLeft,
    selectGroupProduction
  },
  mounted () {
    this.newCreat = this.$route.params.id
    if (this.$route.params.id == 'newCreat') {
    } else {
      this.method = 'put'
      this.getGroupInfo()
    }
  },
  methods: {
    // 获取商品所有规格信息
    getGoodsId (value) {
      groupGoodSku({goods_id: value}).then(res => {
        if (res.status == 200) {
          var goods = res.data
          var arr = []
          goods.forEach((v) => {
            v.prices = ''
            v.stock_counts = ''
            v.total_stock_count = v.stock_count + v.stock_counts
            arr.push(v)
          })
          this.goods = arr
        }
      })
    },
    addGroupSku () {
      // let id = this.goods[0].goods_id
      groupGoodSku({goods_id: this.productId}).then(res => {
        if (res.status == 200) {
          var goodsku = res.data
          var arr = []
          for (var i = 0; i < this.goods.length; i++) {
            for (var j = 0; j < goodsku.length; j++) {
              if (goodsku[j].id == this.goods[i].goods_sku_id) {
                goodsku.splice(j, 1)
                j = j - 1
              }
            }
          }
          // console.log(goodsku)
          goodsku.forEach((v) => {
            v.prices = ''
            v.stock_counts = ''
            v.total_stock_count = v.stock_count + v.stock_counts
            arr.push(v)
          })
          this.goods = this.goods.concat(goodsku)
        }
      })
    },
    // 商品规格
    showSpecs (value) {
      if (value.spec_c) {
        return value.spec_a + ':' + value.property_a + ',' + value.spec_b + ':' + value.property_b + ',' + value.spec_c + ':' + value.property_c + ';'
      } else if (value.spec_b) {
        return value.spec_a + ':' + value.property_a + ',' + value.spec_b + ':' + value.property_b + ';'
      } else if (value.spec_a) {
        return value.spec_a + ':' + value.property_a + ';'
      }
    },
    // 获取编辑的拼团信息
    getGroupInfo () {
      getGroupInfo(this.$route.params.id).then(res => {
        if (res.status == 200) {
          this.ruleForm.exemption = res.data.is_owner_free
          this.ruleForm.outTime = res.data.expire
          this.ruleForm.groupNum = res.data.min_join_count
          this.ruleForm.limitNum = res.data.buy_limit_count
          var start = new Date(res.data.begin_at).getTime()
          var end = new Date(res.data.end_at).getTime()
          this.ruleForm.rangeTime = [start, end]
          var goods = res.data.goods_sku
          goods.forEach(v => {
            v.prices = parseFloat(v.prices / 100).toFixed(2)
            v.total_stock_count = v.stock_count + v.stock_counts
            this.goods.push(v)
          })
          this.productId = this.goods[0].goods_id
        }
      })
    },
    // 批量修改
    groupUpdate () {
      if (this.multipleSelection.length > 0) {
        this.isGroupSet = true
      } else {
        this.$message({
          type: 'warning',
          message: `请选择规格`
        })
      }
    },
    timestampToTime (timestamp) {
      var date = new Date(timestamp)
      var Y = date.getFullYear()
      var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1)
      var D = date.getDate() < 10 ? '0' + date.getDate() : date.getDate()
      var h = date.getHours() < 10 ? '0' + date.getHours() : date.getHours()
      var m = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()
      var s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds()
      return Y + '-' + M + '-' + D + ' ' + h + ':' + m + ':' + s
    },
    // 批量删除
    groupDelete () {
      var newArr = this.goods
      if (this.multipleSelection.length > 0) {
        this.$confirm(`是否删除该规格`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 删除该规格拼团
          for (var i = 0; i < this.goods.length; i++) {
            for (var j = 0; j < this.multipleSelection.length; j++) {
              if (this.goods[i].id == this.multipleSelection[j].id) {
                newArr.splice(i, 1)
              }
            }
          }
          this.goods = newArr
          this.$message({
            type: 'success',
            message: `删除成功`
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: `已取消`
          })
        })
      } else {
        this.$message({
          type: 'warning',
          message: `请选择规格`
        })
      }
    },
    // 设置拼团价格
    setGroupMoney (index) {
      if (this.goods[index].price <= parseFloat(this.goods[index].prices * 100)) {
        this.goods[index].prices = ''
        this.$message({
          message: '拼团价格不能大于等于商品价格',
          type: 'error'
        })
      } else if (isNaN(this.goods[index].prices)) {
        this.goods[index].prices = ''
        this.$message({
          message: '拼团价格只能是数字',
          type: 'error'
        })
      } else if (this.goods[index].prices <= 0) {
        this.goods[index].prices = ''
        this.$message({
          message: '拼团价格必须大于0',
          type: 'error'
        })
      }
    },
    // 设置拼团库存
    setGroupCount (index) {
      if (this.goods[index].total_stock_count < parseFloat(this.goods[index].stock_counts)) {
        this.goods[index].stock_counts = ''
        this.$message({
          message: '拼团库存不能大于商品库存',
          type: 'error'
        })
      } else if (isNaN(this.goods[index].stock_counts)) {
        this.goods[index].stock_counts = ''
        this.$message({
          message: '拼团库存只能是数字',
          type: 'error'
        })
      } else if (this.goods[index].stock_counts < 2) {
        this.goods[index].stock_counts = ''
        this.$message({
          message: '拼团库存必须大于等于2',
          type: 'error'
        })
      } else if (this.goods[index].stock_counts.indexOf('.') > -1) {
        this.goods[index].stock_counts = ''
        this.$message({
          message: '拼团库存必须是整数',
          type: 'error'
        })
      }
    },
    // 删除尺寸
    deleteSku (value) {
      this.$confirm(`是否删除该规格`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 删除该规格拼团
        this.goods.splice(value, 1)
        this.$message({
          type: 'success',
          message: `删除成功`
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: `已取消`
        })
      })
    },
    // 添加商品
    addGroupBuy () {
      this.goodsDialogVisible = true
    },
    handleSelectionChange (val) {
      if (val.length == 0) {
        this.isGroupSet = false
      }
      this.multipleSelection = val
    },
    // 创建拼团
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          var timestart = this.timestampToTime(this.ruleForm.rangeTime[0])
          var timeend = this.timestampToTime(this.ruleForm.rangeTime[1])
          this.ruleForm.obj = {
            is_owner_free: this.ruleForm.exemption,
            begin_at: timestart,
            end_at: timeend,
            expire: this.ruleForm.outTime,
            min_join_count: this.ruleForm.groupNum,
            buy_limit_count: this.ruleForm.limitNum
          }
          // 商品为空
          if (this.goods.length == 0) {
            this.$message({
              message: '请添加商品',
              type: 'error'
            })
          } else {
            var arr = []
            var isTrue = true
            if (this.$route.params.id == 'newCreat') {
              this.goods.forEach((v) => {
                // 没设置商品拼团价格或库存
                if (!v.prices || !v.stock_counts) {
                  isTrue = false
                } else {
                  var o = {}
                  o.sku_id = v.id
                  o.price = v.prices * 100
                  o.goods_id = v.goods_id
                  o.stock_count = v.stock_counts
                  arr.push(o)
                }
              })
              if (isTrue) {
                this.ruleForm.obj.goods_sku = arr
                setGroupInfo(this.ruleForm.obj, this.method).then(res => {
                  if (res.status == 200) {
                    this.$message({
                      type: 'success',
                      message: `拼团成功`
                    })
                    this.$router.push({name: 'groupBuyManagement'})
                  }
                })
              } else {
                this.$message({
                  message: '请正确设置所有商品拼团价格或库存',
                  type: 'error'
                })
              }
            } else {
              // 编辑团购
              this.goods.forEach((v) => {
                // 没设置商品拼团价格或库存
                if (!v.prices || !v.stock_counts) {
                  isTrue = false
                } else {
                  var o = {}
                  o.sku_id = v.id
                  o.prices = v.prices * 100
                  o.goods_id = v.goods_id
                  o.stock_counts = v.stock_counts
                  arr.push(o)
                }
              })
              if (isTrue) {
                this.ruleForm.obj.goods_sku = arr
                // this.ruleForm.obj.goods_groupon_id = this.$route.params.id
                updateGroupInfo(this.ruleForm.obj, this.$route.params.id).then(res => {
                  if (res.status == 200) {
                    this.$message({
                      type: 'success',
                      message: `编辑成功`
                    })
                    this.$router.push({name: 'groupBuyManagement'})
                  }
                })
              } else {
                this.$message({
                  message: '请正确设置所有商品拼团价格或库存',
                  type: 'error'
                })
              }
            }
          }
        } else {
          this.$message({
            message: '设置失败，请认真检查设置项哦',
            type: 'error'
          })
        }
      })
    },
    // 初始化表格选中状态
    toggleSelection (rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    // 批量数据修改
    submitGroupForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          for (var i = 0; i < this.goods.length; i++) {
            for (var j = 0; j < this.multipleSelection.length; j++) {
              if (this.goods[i].id == this.multipleSelection[j].id) {
                this.goods[i].prices = this.setPrice.groupPrice
                this.goods[i].stock_counts = this.setPrice.totalCount
              }
            }
          }
          this.toggleSelection()
          this.isGroupSet = false
          this.$message({
            message: '批量设置设置成功',
            type: 'success'
          })
        } else {
          this.$message({
            message: '设置失败，请认真检查设置项哦',
            type: 'error'
          })
        }
      })
    },
    getHandleClose (msg) {
      this.goodsDialogVisible = msg
    }
  }
}
</script>
<style lang="less">
  .groupNum .el-input__inner,.outTime .el-input__inner,.limitNum .el-input__inner{
    padding-right: 100px;
  }
  .datalist{
    .el-table--scrollable-x .el-table__body-wrapper {
      overflow-x: hidden;
    }
    .el-table th .el-checkbox::after {
      content: '全选';
      color: #333;
      padding-left: 6px;
    }
  }
  .groupbuy-setting-list{
    .el-form-item.is-success .el-input__inner {
      border-color: @green;
    }
    .el-form-item{
      position: relative;
      .tips{
        position: absolute;
        left: 380px;
        color: #999;
        top: 0px;
      }
    }
    .el-date-editor .el-range-input {
      width:100%;
    }
  }
  .datalist {
    padding: 0 25px 20px;
    overflow: hidden;
    background: #fff;
    .el-table th > .cell{
      text-align: center;
    }
    .setGroup{
      background: #fff;
      padding: 20px 0;
     button{
       width: 80px;
       height: 30px;
       padding: 0;
       line-height: 30px;
       text-align: center;
       font-family: MicrosoftYaHei;
       font-size: 12px;
       background: #fff;
       /*color: #333333;*/
       border: 1px solid #dcdfe6;
       border-radius: 2px;
       cursor: pointer;
     }
      button:nth-child(2){
        margin-left: 10px;
      }
      button:nth-child(3){
        border: 1px solid #FA505D;
        color: #FA505D;
      }
      .el-button--default {
        color: #333;
        border-color: #333;
        background: #fff;
        &.is-disabled {
          border: 1px solid #B5B5B5;
          color: #999;
        }
      }
      .el-button--text {
        width: auto;
        padding: 0 7px;
        height: 24px;
        border: 1px solid #63A4FF;
        &:disabled {
          color: @b9;
          border-color: @b5b5;
        }
      }
    }
  }
  .el-tooltip, .cell{
    text-align: center;
  }
  .goods-info-box {
    text-align: left;
    font-size: 0;
    .goods-img {
      width: 60px;
      height: 60px;
      display: inline-block;
      vertical-align: middle;
      text-align: center;
      border: 1px solid #d5d5d5;
      img {
        width: auto;
        height: auto;
        max-width: 100%;
        max-height: 100%;
        position: relative;
        top: 50%;
        transform: translateY(-50%);
      }
    }
    .goods-info {
      display: inline-block;
      vertical-align: middle;
      text-align: left;
      font-size: 12px;
      padding-left: 15px;
      width:calc(100% - 65px);
      .goods-info-name {
        color: #333;
        font-size: 14px;
        line-height: 1.3;
        overflow:hidden;
        text-overflow:ellipsis;
        white-space:nowrap;
      }
      .goods-info-price-category {
        padding-top: 10px;
        line-height: 1.3;
        .goods-info-category {
          color: #999;
          padding-right: 15px;
        }
        .goods-info-price {
          color: #DE5B67;
          display: inline-block;
        }
      }
    }
  }
  .groupbuy-setting-list{
    .el-input__inner{
      width: 370px;
      border-radius:0;
    }
    .groupPrice{
      position: relative;
      .el-input__inner{
        padding-left: 30px;
        position: relative;
      }
    }
    .groupPrice::before{
      position: absolute;
      content: '\FFE5';
      display: block;
      color: #000;
      z-index: 500;
      top: 2px;
      left: 10px;
    }
    .title{
      font-family: MicrosoftYaHei;
      font-size: 14px;
      color: #222222;
      line-height: 17px;
      padding-left: 10px;
      position: relative;
      height: 57px;
      line-height: 57px;
      border-bottom: 1px solid #D5D5D5;
    }
    .el-form-item:first-child {
      margin-top: 10px;
    }
    .title::before{
      display: block;
      content: '';
      position: absolute;
      top:50%;
      transform: translateY(-50%);
      left:0px;
      width: 3px;
      height: 13px;
      background: #FA505D;
    }
    .el-form-item__label{
      text-align: left;
      position: relative;
      font-family: MicrosoftYaHei;
      font-size: 12px;
      color: #3A3A3A;
    }
    .el-radio__input.is-checked .el-radio__inner {
      border-color: #FA505D;
      background: #FA505D;
    }
    .el-radio__input.is-checked + .el-radio__label {
      color: #FA505D;
    }
    .el-form-item__label:before{
      content: '*';
      color: #DE5B67;
      margin-left: -10px;
      padding-right: 5px;
    }
    .el-button--primary {
      color: #fff;
      background-color: #FA505D;
      border-color: #FA505D;
      padding: 9px 15px;
      font-size: 12px;
      width: 80px;
      height: 30px;
      border-radius: 2px;
    }
    .el-button--primary:hover, .el-button--primary:focus {
      background: #fb737d;
      border-color: #fb737d;
      color: #fff; }
  }
</style>
<style scoped lang="less">
    .outTime::after{
      content: '小时';
      position: absolute;
      top: 0px;
      left: 330px;
      color: #999;
    }
    .groupNum::after{
      content: '人';
      position: absolute;
      top: 0px;
      left: 340px;
      color: #999;
    }
    .limitNum::after{
      content: '件';
      position: absolute;
      top: 0px;
      left: 340px;
      color: #999;
    }
    .add-activity-object .el-button--small{
      padding: 0;
      width: 80px;
      height: 30px;
    }
  .delete-btn {
    color: #333;
    padding: 4px 8px;
    border: 1px solid #333;
    font-size: 12px;
    display: block;
    margin: 0 auto;
  }
  .delete-btn:focus{
    color: #333;
    padding: 4px 8px;
    border: 1px solid #333;
    font-size: 12px;
  }
  .delete-btn:hover{
    color: #333;
    padding: 4px 8px;
    border: 1px solid #333;
    font-size: 12px;
  }
  .add-activity-object {
    min-width: 1100px;
    padding-top: 20px;
    margin: 0 20px 0 200px;
    .bread-bar {}
    input {
      color: #333;
      border: 1px solid #d5d5d5;
      display: inline-block;
      width: 236px;
      height: 30px;
      line-height: 30px;
      padding-left: 10px;
      padding-right: 10px;
      &::-webkit-input-placeholder {
        color: #b5b5b5;
      }
      &:disabled {
        background: #d5d5d5;
        border-color: #d5d5d5;
        cursor: not-allowed;
      }
    }
    .flex{
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .plate {
      padding: 20px 20px 20px 15px;
      margin-bottom: 20px;
      background: #fff;
      color: #333;
      font-size: 12px;
      line-height: 15px;
      .required::before {
        content: '*';
        color: #DE5B67;
        margin-left: -10px;
        padding-right: 5px;
      }
      .name {
        padding-right: 3px;
        color: #999;
        width: 145px;
        display: inline-block;
        vertical-align: middle;
      }
      .name.alignment-top {
        vertical-align: top;
        padding-top: 6px;
      }
      .justify {
        text-align: justify;
        &::after {
          content: '';
          padding-left: 100%;
          display: inline-block;
        }
      }
    }
    .groupbuy-setting-list {
      padding: 0 25px 20px;
      li {
        font-size: 12px;
        padding-top: 20px;
        .goods-img-box {
          display: inline-block;
          vertical-align: middle;
          .goods-img {
            display: inline-block;
            vertical-align: middle;
            width: 80px;
            height: 80px;
            position: relative;
            border: 1px solid #d5d5d5;
            text-align: center;
            box-sizing: border-box;
            img {
              display: inline-block;
              width: auto;
              height: auto;
              max-width: 100%;
              max-height: 100%;
              position: relative;
              top: 50%;
              transform: translateY(-50%);
            }
            i {
              font-size: 16px;
              position: absolute;
              background: #ffffff;
              color: #D5D5D5;
              top: -8px;
              right: -8px;
              cursor: pointer;
              border-radius: 50%;
            }
          }
          .select-goods {
            display: inline-block;
            vertical-align: middle;
            box-sizing: border-box;
            font-size: 20px;
            text-align: center;
            line-height: 80px;
            width: 80px;
            height: 80px;
            color: #d5d5d5;
            border: 1px dashed #d5d5d5;
          }
        }
        .goods-price {
          position: relative;
          &::before {
            content: '￥';
            display: block;
            position: absolute;
            top: 2px;
            left: 10px;
          }
          input {
            padding-left: 30px;
            width: 216px;
          }
        }
        .activity-time {
          display: inline-block;
          vertical-align: middle;
          .el-range-editor {
            padding: 0 10px;
            height: 32px;
            border-color: #d5d5d5;
            border-radius: 0;
          }
        }
        .el-button{
          width: 80px;
          height: 30px;
        }
      }
    }
  }
</style>
