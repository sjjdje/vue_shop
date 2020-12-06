<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialogVisible">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table
        class="tree-table"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效的模板 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false"></i>
          <i class="el-icon-error" v-else></i>
        </template>
        <!-- 排序模板 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作模板 -->
        <template slot="opt" slot-scope>
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
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
      ></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      @close="addCateDialogClose"
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="30%"
    >
      <!-- 表单区域 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            :clearable="true"
            :options="parentsList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="parentCateChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 查询商品分类列表所要向服务器端传递的参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      cateList: [],
      // 存储总数据条数
      total: 2,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      //   控制添加分类对话框的显示与影藏
      addCateDialogVisible: false,
      //   添加分类的表单数据对象
      addCateForm: {
        // 父级分类的ID
        cat_pid: 0,
        //   将要添加的分类的名称
        cat_name: '',
        // 父级分类的等级，默认为等级一
        cat_level: 0
      },
      //   添加分类标点的校验规则
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类的名称',
            trigger: 'blur'
          }
        ]
      },
      // 存储添加分类对话框里的父级分类列表
      parentsList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
        expandTrigger: 'hover'
      },
      // 级联选择器的双向绑定数组（选中的父级分类的id
      selectedKeys: []
    }
  },
  created() {
    //   调用商品分类列表函数
    this.getCateList()
  },
  methods: {
    async getCateList() {
      // 获取商品分类数据列表
      //   因为是get请求类型，所以要先加上params再加data里的参数？？？？？？？？？？？
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      //   console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 监听添加分类按钮事件
    showAddCateDialogVisible() {
      // 获取父级的分类数据列表
      this.getParentsCateList()
      this.addCateDialogVisible = true
    },
    // 为添加分类对话框获取所有父级分类的列表
    async getParentsCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('父级分类获取失败')
      }
      // console.log(res.data)
      this.parentsList = res.data
    },
    // 监听级联选择器改变的事件
    parentCateChange() {
      // console.log(this.selectedKeys)
      // 如果selectedKeys数组的长度大于0，证明选中了父级分类，反之就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 监听添加分类对话框的确定按钮事件s
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.getCateList()
        this.$message.success('添加分类成功')
        this.addCateDialogVisible = false
      })
      // console.log(this.addCateForm)
    },
    // 监听添加分类对话框的关闭事件
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>
<style lang="less" scoped>
.el-icon-success {
  color: lightgreen;
}
.el-icon-error {
  color: red;
}
.tree-table {
  margin-top: 15px;
}
.el-pagination {
  margin-top: 10px;
}
.el-cascader {
  width: 100%;
}
</style>
