<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-alert title="添加商品信息"
                type="info"
                center
                show-icon
                :closable="false">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200"
                :active="activeIndex - 0"
                finish-status="success"
                align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px"
               label-position="top">
        <el-tabs tab-position="left"
                 v-model="activeIndex"
                 :before-leave="beforeTabLeave"
                 @tab-click="tabClicked">
          <el-tab-pane label="基本信息"
                       name="0">
            <el-form-item label="商品名称"
                          prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格"
                          prop="goods_price">
              <el-input v-model="addForm.goods_price"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量"
                          prop="goods_weight">
              <el-input v-model="addForm.goods_weight"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量"
                          prop="goods_number">
              <el-input v-model="addForm.goods_number"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类"
                          prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat"
                           :options="cateList"
                           :props="cascaderProps"
                           @change="handleChange"
                           :clearable="true"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数"
                       name="1">
            <!-- 渲染表单的Item项目 -->
            <el-form-item :label="item.attr_name"
                          v-for="item in manyTableData"
                          :key="item.attr_id">
              <!-- 复选框 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb"
                             v-for="(cb,index) in item.attr_vals"
                             :key="index"
                             border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性"
                       name="2">
            <el-form-item :label="item.attr_name"
                          v-for="item in onlyTableData"
                          :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片"
                       name="3">
            <el-upload :action="uploadUrl"
                       :on-preview="handlePreview"
                       :on-remove="handleRemove"
                       list-type="picture"
                       :headers="headerObj"
                       :on-success="handleSuccess">
              <el-button size="small"
                         type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容"
                       name="4">
            <!-- 富文本编辑器 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary"
                       class="btnADD"
                       @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!--图片预览 -->
    <el-dialog title="图片预览"
               :visible.sync="preDialogVisible"
               width="50%">
      <img :src="previewUrl"
           alt=""
           class="preViewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'Add',
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属分类id数组
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      cateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTableData: [],
      onlyTableData: [],
      uploadUrl: 'http://timemeetyou.com:8889/api/private/v1/upload',
      previewUrl: '',
      // 图片上传组件的headers
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      preDialogVisible: false
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data } = await this.$http.get('categories')
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.cateList = data.data
    },
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择分类')
        return false
      }
    },
    async tabClicked() {
      // 访问的动态参数面板
      if (this.activeIndex === '1') {
        const { data } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (data.meta.status !== 200) {
          return this.$message.error('获取动态参数失败')
        }
        data.data.forEach(element => {
          element.attr_vals = element.attr_vals.length === 0 ? [] : element.attr_vals.split(' ')
        })
        this.manyTableData = data.data
      } else if (this.activeIndex === '2') {
        const { data } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (data.meta.status !== 200) {
          return this.$message.error('获取静态属性失败')
        }
        this.onlyTableData = data.data
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      this.previewUrl = file.response.data.url
      this.preDialogVisible = true
    },
    // 处理移除图片的参数
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(item => item.pic === filePath)
      this.addForm.pics.splice(index, 1)
    },
    // 图片上传成功
    handleSuccess(response) {
      // 1拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2将图片信息对象存入pics数组中
      this.addForm.pics.push(picInfo)
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的信息')
        }
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        const { data } = await this.$http.post('goods', form)
        if (data.meta.status !== 201) {
          this.$message.error('添加商品失败')
        }
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  }
}
</script>

<style lang="less" scope>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.preViewImg {
  width: 100%;
}

.btnADD {
  margin-top: 15px !important;
}
</style>
