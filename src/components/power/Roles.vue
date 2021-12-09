<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 table表格-->
      <el-table :data="rolesList" stripe border>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot:default="{ row }">
            <!-- 渲染一级权限 -->
            <el-row
              v-for="(item1, i1) in row.children"
              :key="item1.id"
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter', 'expandMargin']"
            >
              <el-col :span="5">
                <el-tag closable @close="removeRightById(row, item1.id)">{{
                  item1.authName
                }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                >
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(row, item2.id)">{{
                      item2.authName
                    }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="285px">
          <template v-slot:default="{ row }">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="rolesdelete(row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClose"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        default-expand-all
        node-key="id"
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDislogClosed">
      <el-form
        :model="addRolesForm"
        :rules="addFormRules"
        ref="addRolesForm"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRolesUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @closed="editDialogClosed"
    >
      <el-form
        :model="editRolesForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editFormInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      rolesList: [],
      // 分配权限的对话框开关
      setRightDialogVisible: false,
      rightsList: [],
      // 树形控件绑定的对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的对象
      defKeys: [],
      // 角色id
      roleId: '',
      // 添加角色对话框开关
      addDialogVisible: false,
      // 添加角色的配置
      addRolesForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名字', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '输入的范围是 3 ~ 10 为字符',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          {
            min: 5,
            max: 20,
            message: '输入的范围是 5 ~ 20 为字符',
            trigger: 'blur'
          }
        ]
      },
      // 编辑角色对话框开关
      editDialogVisible: false,
      editRolesForm: {},
      // 编辑角色的规则
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名字', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '输入的范围是 3 ~ 10 为字符',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          {
            min: 5,
            max: 20,
            message: '输入的范围是 5 ~ 20 为字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.rolesList = res.data
    },
    // 根据id删除权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消了该操作')
      // 返回当前角色下最新权限
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      // 单独刷新展开列
      role.children = res.data
    },
    // 打开权限对话框
    async showSetRightDialog(role) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限数据失败')
      this.rightsList = res.data
      // 获取三级权限id
      this.getLeafKeys(role, this.defKeys)
      // 赋值roleId，方便调用
      this.roleId = role.id
      this.setRightDialogVisible = true
    },
    // 递归调用，得到三级权限id并保存到defKeys
    getLeafKeys(node, arr) {
      if (!node.children) return arr.push(node.id)
      // 遍历childern的数组，再次调用递归
      node.children.forEach((item) => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭
    setRightDialogClose() {
      // 清空defKeys
      this.defKeys = []
    },
    // 确定提交分配的权限
    async allotRights() {
      const key = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = key.join()
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 关闭角色对话框的回调
    addDislogClosed() {
      this.$refs.addRolesForm.resetFields()
    },
    // 添加角色
    addRolesUser() {
      this.$refs.addRolesForm.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败!')
        }
        this.$message.success('添加角色成功!')
        this.getRolesList()
        this.addDialogVisible = false
      })
    },
    // 打开编辑角色的会话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败!')
      }
      this.editRolesForm = res.data
      this.editDialogVisible = true
    },
    // 关闭编辑角色对话框的回调
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑角色
    editFormInfo() {
      this.$refs.editFormRef.validate((valid) => {
        if (!valid) return
        this.$http
          .put('roles/' + this.editRolesForm.roleId, {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc
          })
          .then((res) => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('修改角色失败!')
            }
            this.$message.success('修改角色成功!')
            this.getRolesList()
          })
        this.editDialogVisible = false
      })
    },
    // 删除角色
    async rolesdelete(id) {
      const confirmRusult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '删除角色',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmRusult !== 'confirm') {
        return this.$message.info('已取消该删除操作')
      }
      this.$http.delete('roles/' + id).then((res) => {
        const { data: response } = res
        if (response.meta.status !== 200) {
          return this.$message.error('该用户删除失败')
        }
        this.$message.success('该用户已经删除')
        this.getRolesList()
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
// 垂直方向居中
.vcenter {
  display: flex;
  align-items: center;
}
</style>
