<template>
  <div>
    <!-- 面包屑区域 -->
    <Breadcrumb name1="权限管理" name2="权限列表"></Breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <!-- 权限管理列表区域,table表格 -->
      <el-table :data="rigthsList" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级">
          <template v-slot:default="{ row }">
            <el-tag v-if="row.level === '0'">一级</el-tag>
            <el-tag type="success" v-if="row.level === '1'">二级</el-tag>
            <el-tag type="warning" v-if="row.level === '2'">三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import Breadcrumb from '../content/breadcrumb/Breadcrumb.vue'

export default {
  name: 'Rights',
  data() {
    return {
      rigthsList: []
    }
  },
  created() {
    // 调用用户权限管理列表
    this.getRightsList()
  },
  components: { Breadcrumb },
  methods: {
    // 获取权限管理列表
    async getRightsList() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取权限管理列表失败')
      this.rigthsList = res.data
    }
  }
}
</script>

<style lang="less" scoped></style>
