<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }" @click.native="clearStorage">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input
          placeholder="请输入搜索内容"
          prefix-icon="el-icon-search"
          v-model="queryInfo.query"
          clearable
          @clear="getGoodsList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="goAddGoodsPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="goodsList" border style="width: 100%">
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称" ></el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="60"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="60"></el-table-column>
        <el-table-column label="创建时间" width="200">
          <template slot-scope="scope">
            {{scope.row.add_time | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="openEditGoods(scope.row)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteGoods(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页功能 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
        >
      </el-pagination>
      <!-- 修改商品信息 -->
      <el-dialog
        title="修改商品信息"
        :visible.sync="editGoodsVisible"
        width="50%"
        @close="resetEditForm"
        ref="editRefs"
      >
        <el-form label-width="80px" :model="editGoodsForm" ref="formRefs" :rules="editFormRules">
          <el-form-item label="商品名称" prop="goods_name">
            <el-input v-model="editGoodsForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="价格" prop="goods_price">
            <el-input v-model="editGoodsForm.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="数量" prop="goods_number">
            <el-input v-model="editGoodsForm.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="重量" prop="goods_weight">
            <el-input v-model="editGoodsForm.goods_weight"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editGoodsVisible = false">取 消</el-button>
          <el-button type="primary" @click="editGoods">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      editGoodsVisible: false,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      goodsList: [],
      selectGoodsId: '',
      editGoodsForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        goods_introduce: '',
        pics: [],
        attrs: []
      },
      editFormRules: {
        goods_name: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    clearStorage () {
      window.sessionStorage.setItem('activePath', '')
      this.$router.go(0)
    },
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表数据失败')
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    deleteGoods (id) {
      this.$confirm('此操作将永久删除该信息, 是否继续?', '删除', {
        confirmButtonText: '删除',
        cancelButtonText: '取消',
        type: 'error'
      }).then(async () => {
        const { data: res } = await this.$http.delete('goods/' + id)
        if (res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.getGoodsList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    resetEditForm () {
      this.$refs.formRefs.resetFields()
    },
    openEditGoods (row) {
      this.editGoodsVisible = true
      this.selectGoodsId = row.goods_id
      this.editGoodsForm.goods_name = row.goods_name
      this.editGoodsForm.goods_price = row.goods_price
      this.editGoodsForm.goods_number = row.goods_number
      this.editGoodsForm.goods_weight = row.goods_weight
    },
    editGoods () {
      this.$refs.formRefs.validate(async (valid) => {
        if (!valid) return this.$message.error('提交信息有误')
        const { data: res } = await this.$http.put('goods/' + this.selectGoodsId, this.editGoodsForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success('修改成功')
        this.getGoodsList()
        this.editGoodsVisible = false
      })
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    goAddGoodsPage () {
      this.$router.push('/goods/addGoods')
    }
  }
}
</script>

<style>

</style>
