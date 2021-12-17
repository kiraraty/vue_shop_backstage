<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 表格占位 -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :expand-type="false"
        :selection-type="false"
        show-index
        index-text="#"
        stripe
        border
        :show-row-hover="false"
        :columns="columns"
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <template slot="option" slot-scope="scope">
          <el-button
            type="primary"
            @click="showeditCateDialog(scope.row)"
            icon="el-icon-edit"
            size="mini"
            >编辑</el-button
          >
          <el-button
            type="danger"
            @click="removeCate(scope.row.cat_id)"
            icon="el-icon-delete"
            size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类表单 -->
      <span slot="footer" class="dialog-footer">
        <el-form
          :model="addCateForm"
          :rules="addCateFormRules"
          ref="addCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称:" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>

          <el-form-item label="父级分类">
          <!-- options指定数据源 props用来指定配置对象 v-model指定数组类型-->
              <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{
              expandTrigger: 'hover',
              ...cascaderProps,
              checkStrictly: 'true'
            }"
            @change="parentCateChange"
            clearable
            change-on-select="true"
            class="cascader-form"
          ></el-cascader>
          </el-form-item>
        </el-form>
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类的数据列表,默认为空
      cateList: [],
      // 请求参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          type: "template",
          template: "isok",
        },
        {
          label: "排序",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "option",
        },
      ],
      addCateDialogVisible: false,
      //添加分类表单对象
      addCateForm: {
        cat_name: "",
        cat_pid: "",
        //分类的等级
        cat_level: "",
      },
      //添加分类表达验证对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },
      parentCateList: [],
      //级联选择器对象
      cascaderProps:{
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      //选中的父级分类的id数组
      selectedKeys:[]
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    //获取商品数据分类列表
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败!");
      }
      // 把数据列表赋值给cateList
      this.cateList = res.data.result;
      // 获取商品的总数量
      this.total = res.data.total;
      console.log(res);
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    showAddCateDialog() {
      //获取父级数据列表
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类数据失败!");
      }
      console.log(res.data);
      this.parentCateList = res.data;
    },
    parentCateChange(){
      
      //如果selected里面的length>0证明选中了父级分类
      if(this.selectedKeys.length>0)
      {//父级分类的id
       this.addCateForm.cat_pid=this.selectedKeys[this.selectedKeys.length-1];
       //分类的等级
       this.addCateForm.cat_level=this.selectedKeys.length;
       return;
      }
      else{
        //父级分类的id
       this.addCateForm.cat_pid=0;
       //分类的等级
       this.addCateForm.cat_level=0;
      }

    },
    addCate(){
      this.$refs.addCateFormRef.validate( async valid=>{
        if(!valid)return ;
   const{data:res} =  await this.$http.post('categories',this.addCateForm);
   if(res.meta.status!==201){
     return this.$message.error("添加分类失败");
   }
   this.$message.success('添加分类成功！');
   this.getCateList();
   this.addCateDialogVisible=false;
   
      });
    },
    //监听对话框的关闭事件
    addCateDialogClosed(){
        this.$refs.addCateFormRef.resetFields();
        this.selectedKeys=[];
        this.addCateForm.cat_level=0;
        this.addCateForm.cat_pid=0;
    }
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.cascader-form{
  width:100%;
}
</style>
