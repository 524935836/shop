<template>
  <div>
    <!-- 面包屑区域 -->
    <Breadcrumb name1="商品管理" name2="参数列表"></Breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert title="警告提示的文案" type="warning" :closable="false" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col :span="3">
          <span>选择商品分类：</span>
        </el-col>
        <el-col :span="21">
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectKey"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab标签页区域 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <!-- 添加动态属性的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            size="mini"
            :disabled="isBtnDisabled"
            type="primary"
            @click="addParamsDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template v-slot:default="{ row }">
                <!-- tag标签 -->
                <el-tag
                  v-for="(item, i) in row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, row)"
                  >{{ item }}</el-tag
                >
                <!-- 文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="row.inputVisible"
                  v-model="row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(row)"
                  @blur="handleInputConfirm(row)"
                >
                </el-input>
                <!-- 按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot:default="{ row }">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditParamsDialog(row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态参数" name="only">
          <el-button
            size="mini"
            :disabled="isBtnDisabled"
            type="primary"
            @click="addParamsDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 静态参数表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template v-slot:default="{ row }">
                <!-- tag标签 -->
                <el-tag
                  v-for="(item, i) in row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i, row)"
                  >{{ item }}</el-tag
                >
                <!-- 文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="row.inputVisible"
                  v-model="row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(row)"
                  @blur="handleInputConfirm(row)"
                >
                </el-input>
                <!-- 按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template v-slot:default="{ row }">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditParamsDialog(row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addParamsDialogVisible"
      width="50%"
      @close="addParamsDialogClosed"
    >
      <!-- 添加参数的表单 -->
      <el-form
        :model="addParamsForm"
        :rules="addParamsFormRules"
        ref="addParamsFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editParamsDialogVisible"
      width="50%"
      @close="editParamsDialogClosed"
    >
      <!-- 修改参数的表单 -->
      <el-form
        :model="editParamsForm"
        :rules="editParamsFormRules"
        ref="editParamsFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import Breadcrumb from '../content/breadcrumb/Breadcrumb.vue'

export default {
  name: 'Params',
  data() {
    return {
      cateList: [],
      cateProps: {
        expandTrigger: 'hover',
        // v-model绑定value，作为selectKey数组中的id
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectKey: [],
      // tab页签双向绑定的数据
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      // 添加参数对话框的开关
      addParamsDialogVisible: false,
      // 添加表单的属性
      addParamsForm: {
        attr_name: ''
      },
      addParamsFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      // 修改对话框的开关
      editParamsDialogVisible: false,
      // 修改表单的属性
      editParamsForm: {},
      editParamsFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  components: { Breadcrumb },
  methods: {
    // 获取所有商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.cateList = res.data
    },
    // 获取参数列表
    async getParamsList() {
      if (this.selectKey.length !== 3) return
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: {
          sel: this.activeName
        }
      })
      if (res.meta.status !== 200) return this.$message.error('获取参数列表失败')
      // 将数据中的attr_vals字符串拆分成数组
      res.data.forEach((item) => {
        // ''.split(' ') === ['']
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 切换按钮和文本框的开关
        item.inputVisible = false
        // 双向绑定文本框内容
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 级联选择器子节点变化时的回调
    handleChange() {
      this.manyTableData = []
      this.onlyTableData = []
      // 判断是否是三级分类
      if (this.selectKey.length !== 3) {
        this.selectKey = []
        return
      }
      this.getParamsList()
    },
    // tab标签页发生变化时的回调
    handleClick() {
      this.getParamsList()
    },
    // 添加参数对话框关闭的回调
    addParamsDialogClosed() {
      this.$refs.addParamsFormRef.resetFields()
    },
    // 点击确定添加参数
    addParams() {
      this.$refs.addParamsFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.getParamsList()
        this.$message.success('添加参数成功')
        this.addParamsDialogVisible = false
      })
    },
    // 显示修参数的改对话框并获取参数信息
    async showEditParamsDialog(attrId) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrId}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) return this.$message.error('获取参数信息失败')
      this.editParamsForm = res.data
      this.editParamsDialogVisible = true
    },
    // 修改参数对话框关闭的回调
    editParamsDialogClosed() {
      this.$refs.editParamsFormRef.resetFields()
    },
    // 点击确定修改参数
    editParams() {
      this.$refs.editParamsFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`,
          { attr_sel: this.activeName, attr_name: this.editParamsForm.attr_name }
        )
        if (res.meta.status !== 200) return this.$message.error('修改参数失败')
        this.getParamsList()
        this.$message.success('修改参数成功')
        this.editParamsDialogVisible = false
      })
    },
    // 删除参数
    async removeParams(attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrid}`
      )
      if (res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.getParamsList()
      this.$message.success('删除参数成功')
    },
    // 文本框按下enter或失去焦点时触发
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // push进用来遍历的attr_vals
      row.attr_vals.push(row.inputValue.trim())
      // 清空inputValue，enter触发keyup判断为空，直接return出去
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    // 发起请求，编辑提交的参数项
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: this.activeName,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) return this.$message.error('修改参数项失败')
      this.$message.success('修改参数项成功')
    },
    // 点击按钮展示文本框
    showInput(row) {
      // 显示文本框
      row.inputVisible = true
      // 在下一次DOM更新结束后执行其指定的回调
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除参数项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.selectKey.length !== 3) return true
      return false
    },
    // 分类id
    cateId() {
      if (this.selectKey.length === 3) return this.selectKey[2]
      return null
    },
    // 添加参数对话框标题
    titleText() {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  display: flex;
  align-items: center;
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 100px;
}
</style>
