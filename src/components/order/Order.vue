<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append"
                       icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表数据 -->
      <el-table :data="orderList"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号"
                         prop="order_number"></el-table-column>
        <el-table-column label="订单价格"
                         prop="order_price"></el-table-column>
        <el-table-column label="是否付款"
                         prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success"
                    v-if="scope.row.pay_status ==='1'">已付款</el-tag>
            <el-tag type="danger"
                    v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货"
                         prop="is_send"></el-table-column>
        <el-table-column label="下单时间"
                         prop="create_time">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="showBox"></el-button>
            <el-button type="success"
                       icon="el-icon-location"
                       size="mini"
                       @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 7, 10]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
      <!-- 修改地址对话框 -->
      <el-dialog title="提示"
                 :visible.sync="addressDialogVisible"
                 width="50%"
                 @close="addressDialogClosed">
        <el-form ref="addressFormRef"
                 :model="addressForm"
                 label-width="100px"
                 :rules="addressFormRules">
          <el-form-item label="省市区/县"
                        prop="address1">
            <el-cascader :options="cityData"
                         v-model="addressForm.address1"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址"
                        prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="addressDialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="addressDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 展示物流进度 -->
      <el-dialog title="提示"
                 :visible.sync="progressDialogVisible"
                 width="50%">
        <span>物流信息接口失效</span>
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="progressDialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="progressDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './cityData'

export default {
  name: 'Order',
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      total: 0,
      orderList: [],
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [{ required: true, message: '请选择省市区', trigger: 'blur' }],
        address2: [{ required: true, message: '请填写详细地址', trigger: 'blur' }]
      },
      cityData,
      progressDialogVisible: false,
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (data.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.total = data.data.total
      this.orderList = data.data.goods
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showBox() {
      this.addressDialogVisible = true
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox() {
      const { data } = await this.$http.get('kuaidi/804909574412544580')
      this.progressDialogVisible = true
      if (data.meta.status !== 200) {
        return this.$message.error('获取物流信息失败')
      }
      this.progressInfo = data.data
    }
  }
}
</script>

<style lang="less" scope>
</style>
