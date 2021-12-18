<template>
  <div>
    <!-- 面包屑区域 -->
    <Breadcrumb name1="商品管理" name2="商品列表"></Breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 输入框 -->
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格 -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column
          prop="goods_price"
          label="商品价格(元)"
          width="95px"
        ></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="70px"></el-table-column>
        <el-table-column label="创建时间" width="150px">
          <template v-slot:default="{ row }">
            {{ row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot="{ row }">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>
    <!-- 编辑对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @closed="editDialogClosed"
    >
      <el-form
        :model="editGoodsForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editGoodsForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="editGoodsForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number">
          <el-input v-model="editGoodsForm.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editGoodsForm.goods_weight"></el-input>
        </el-form-item>
        <!-- 级联选择器 -->
        <el-form-item label="商品分类" prop="goods_cat">
          <el-cascader
            v-model="editGoodsForm.goods_cat"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
            ref="cascaderRef"
          >
          </el-cascader>
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
import _ from 'lodash'
import Breadcrumb from '../content/breadcrumb/Breadcrumb.vue'

export default {
  name: 'GoodsList',
  data() {
    return {
      cateList: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0,
      // 编辑商品的对话框开关
      editDialogVisible: false,
      editGoodsForm: {},
      editFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      // 级联选择器配置
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  components: { Breadcrumb },
  methods: {
    // 获取商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品列表失败')
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    // 单页项目个数变化的回调
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 页码发生变化的回调
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    // 删除商品
    async removeById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消删除商品')
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除商品失败')
      this.getGoodsList()
      this.$message.success('删除商品成功')
    },
    // 显示编辑商品对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败!')
      }
      this.getCateList()
      this.editGoodsForm = res.data
      this.editGoodsForm.goods_cat =
        this.editGoodsForm.goods_cat.length === 0
          ? []
          : this.editGoodsForm.goods_cat.split(',').map((item) => item * 1)
      this.$nextTick(($) => {
        this.editDialogVisible = true
      })
    },
    // 关闭编辑商品对话框的回调
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑商品
    editFormInfo() {
      this.$refs.editFormRef.validate((valid) => {
        if (!valid) return
        // 深度拷贝addForm对象
        const form = _.cloneDeep(this.editGoodsForm)
        // 将分类id数组转为字符串
        form.goods_cat = form.goods_cat.join()
        this.$http
          .put('goods/' + this.editGoodsForm.goods_id, {
            goods_name: this.editGoodsForm.goods_name,
            goods_price: this.editGoodsForm.goods_price,
            goods_number: this.editGoodsForm.goods_number,
            goods_weight: this.editGoodsForm.goods_weight,
            goods_cat: form.goods_cat
          })
          .then((res) => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('修改商品失败!')
            }
            this.$message.success('修改商品成功!')
            this.getGoodsList()
          })
        this.editDialogVisible = false
      })
    },
    // 获取商品全部分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.cateList = res.data
    },
    // 级联选择器发生变化的回调
    handleChange() {
      if (this.editGoodsForm.goods_cat.length !== 3) return (this.editGoodsForm.goods_cat = [])
    },
    // 跳转至商品添加页面
    goAddPage() {
      this.$router.push('goods/add')
    }
  }
}
</script>

<style lang="less" scoped></style>
