<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="dialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="roleList"
                border
                stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item,index) in scope.row.children"
                    :class="['bdbottom',index===0?'bdtop':'','vcenter']"
                    :key="item.id">
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightById(scope.row,item.id)">{{item.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!-- 渲染二级权限 -->
                <el-row v-for="(cItem,cIndex) in item.children"
                        :key="cItem.id"
                        :class="[cIndex === 0?'':'bdtop','vcenter']">
                  <el-col :span="6">
                    <el-tag type="success"
                            closable
                            @close="removeRightById(scope.row,cItem.id)">{{cItem.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="gItem in cItem.children"
                            :key="gItem.id"
                            type="warning"
                            closable
                            @close="removeRightById(scope.row,gItem.id)">{{gItem.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="editDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning"
                       icon="el-icon-setting"
                       size="mini"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="dialogVisible"
               width="30%"
               @close="dialogClosed">
      <el-form ref="addFormRef"
               :model="addForm"
               :rules="addFormRules"
               label-width="80px">
        <el-form-item label="角色名"
                      prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色 -->
    <el-dialog title="修改角色信息"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClosed">
      <el-form ref="editFormRef"
               :model="editForm"
               :rules="editFormRules"
               label-width="80px">
        <el-form-item label="角色ID">
          <el-input v-model="editForm.roleId"
                    :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editRolesInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限 -->
    <el-dialog title="分配权限"
               :visible.sync="setRightDialogVisible"
               width="50%"
               @close="setRightDialogClosed">
      <el-tree :data="rightsList"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      roleList: [],
      // 控制添加角色对话框的显示与隐藏
      dialogVisible: false,
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' }
        ]
      },
      // 编辑角色
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' }
        ]
      },
      // 分配权限
      setRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点的id值数组
      defKeys: [],
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data } = await this.$http.get('roles')
      if (data.meta.status !== 200) {
        return this.$message.error('获取角色列表')
      }
      this.roleList = data.data
    },
    // 添加角色
    dialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data } = await this.$http.post('roles', this.addForm)
        if (data.meta.status !== 201) {
          this.dialogVisible = false
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.dialogVisible = false
        this.getRolesList()
      })
    },
    // 编辑角色的对话框
    async editDialog(id) {
      const { data } = await this.$http.get('roles/' + id)
      if (data.meta.status !== 200) {
        return this.$message.error('查询角色信息失败')
      }
      this.editForm = data.data
      this.editDialogVisible = true
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    editRolesInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        if (data.meta.status !== 200) {
          return this.$message.error('更新角色信息失败')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新角色信息成功')
      })
    },
    // 删除角色
    async removeRoleById(id) {
      const confirmRes = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      if (confirmRes) {
        const { data } = await this.$http.delete('roles/' + id)
        if (data.meta.status !== 200) {
          return this.$message.error('删除角色失败!')
        }
        this.$message.success('删除用户成功')
        this.getRolesList()
      }
    },
    // 根据id删除对应权限
    async removeRightById(role, right) {
      const confirmRes = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      if (confirmRes !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data } = await this.$http.delete(`roles/${role.id}/rights/${right}`)
      if (data.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      this.$message.success('删除权限成功')
      role.children = data.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data } = await this.$http.get('rights/tree')
      if (data.meta.status !== 200) {
        return this.$message.error('获取用户权限失败')
      }
      this.rightsList = data.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归获取所有三级权限的id
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(element => {
        this.getLeafKeys(element, arr)
      })
    },
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (data.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
