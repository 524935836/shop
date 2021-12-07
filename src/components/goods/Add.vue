<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/home/goods' }">商品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <!-- 提示信息 -->
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps
        :space="200"
        :active="activeIndex * 1"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- Form 表单 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <!-- tabs标签页 -->
        <!-- v-model绑定值，选中选项卡的 name -->
        <el-tabs
          tab-position="left"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClick"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <!-- 级联选择器 -->
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
                @visible-change="visibleChange"
                ref="cascaderRef"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染参数列表 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <!-- 渲染属性列表 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- 上传图片 -->
            <el-upload
              :action="upload"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headersObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary" class="uploadBtn"
                >点击上传</el-button
              >
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加按钮 -->
            <el-button type="primary" @click="addGoods" class="addBtn"
              >添加商品</el-button
            >
          </el-tab-pane>
          <!-- 点击上一步 -->
          <el-button @click="preNext('pre')" size="medium" v-show="activeIndex * 1 !== 0"
            >上一步</el-button
          >
          <!-- 点击下一步 -->
          <el-button
            class="buttonNext"
            @click="preNext('next')"
            size="medium"
            v-show="activeIndex * 1 !== 4"
            >下一步</el-button
          >
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览的对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" class="previewImg" />
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
      // 添加商品的表单数据
      addForm: {
        goods_name: '',
        goods_price: '0',
        goods_weight: '0',
        goods_number: '0',
        // 添加商品要转化成字符串
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
      // 级联选择器配置
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 参数列表
      manyTableData: [],
      // 属性列表
      onlyTableData: [],
      // 图片上传地址
      upload: 'http://127.0.0.1:8888/api/private/v1/upload',
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 预览图片的地址
      previewPath: '',
      // 预览图片的开关
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品全部分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.cateList = res.data
    },
    // 级联选择器发生变化的回调
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) return (this.addForm.goods_cat = [])
    },
    // 级联选择器下拉框出现的回调
    visibleChange(value) {
      if (value) {
        // 防止级联选择器抖动
        this.$nextTick((_) => {
          this.$refs.cascaderRef.$refs.popper.className =
            'top el-popper el-cascader__dropdown'
        })
      }
    },
    // tab标签页切换的回调
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        // 三秒提示一次
        if (this.timer) return false
        this.timer = setTimeout(() => {
          this.timer = null
        }, 3000)
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    // tab页被选中时触发
    async tabClick() {
      if (this.activeIndex === '1') {
        // 获取动态参数列表
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'many' } }
        )
        if (res.meta.status !== 200) return this.$message.error('获取动态参数失败')
        res.data.forEach((item) => {
          // 将参数字符串转为参数数组以遍历数据
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        // 获取静态属性列表
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'only' } }
        )
        if (res.meta.status !== 200) return this.$message.error('获取静态属性失败')
        this.onlyTableData = res.data
      }
    },
    // 点击图片的回调
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 删除图片的回调
    handleRemove(file) {
      // 被删除的图片
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex((x) => x.pic === filePath)
      // 删除addForm中的图片路径
      this.addForm.pics.splice(i, 1)
    },
    // 图片上传成功的回调
    handleSuccess(response) {
      // 将图片的地址保存到addForm
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    // 点击调到下一步
    preNext(dir) {
      if (dir === 'next') {
        this.activeIndex++
        this.activeIndex = this.activeIndex + ''
        // activeIndex + 1，警告选择分类后再减1
        this.$nextTick((_) => {
          if (this.addForm.goods_cat.length !== 3) {
            return --this.activeIndex
          }
          this.tabClick()
        })
      } else {
        this.activeIndex--
        this.activeIndex = this.activeIndex + ''
        this.tabClick()
      }
    },
    // 添加商品
    addGoods() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return this.$message.error('请填写必要的表单项')
        // 深度拷贝addForm对象
        const form = _.cloneDeep(this.addForm)
        // 将分类id数组转为字符串
        form.goods_cat = form.goods_cat.join()
        // 处理动态参数
        this.manyTableData.forEach((item) => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach((item) => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // 发送请求添加商品
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        this.$router.push('/home/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less">
.top {
  top: 452px !important;
}
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.buttonNext {
  position: relative;
  left: 82%;
}
.addBtn {
  position: relative;
  left: 90%;
}
.uploadBtn {
  margin-bottom: 130px !important;
}
</style>
