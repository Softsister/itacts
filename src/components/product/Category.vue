<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div>
      <el-button type="success" plain @click='addHandler'>添加分类</el-button>
    </div>
    <div>
      <!-- columns表示所有的列，tree-structure表示数据格式，data-source表示实际的列表数据，deleteCate处理删除操作，showForm处理编辑操作，refresh处理刷新操作 -->
      <tree-grid :columns="columns" :tree-structure="true" :data-source="dataSource" @deleteCate="deleteCategory" @showForm="showEditForm" @refresh="initList"></tree-grid>
    </div>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[5, 10, 50, 100]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <!-- 添加分类弹窗 -->
    <el-dialog
      title="添加分类"
      :visible="dialogVisible4Add"
      width="50%">
      <div>
        <span>分类名称：</span>
        <el-input class='cname' v-model="cate.cat_name"></el-input>
      </div>
      <div>
        <span>父级分类：</span>
        <el-cascader
          :options="cateList"
          v-model="selectedOptions"
          :props='propsDefine'
          :show-all-levels="false"
          @change="handleChange">
        </el-cascader>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible4Add = false">取 消</el-button>
        <el-button type="primary" @click="submitCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类弹窗 -->
    <el-dialog
      title="编辑分类"
      :visible="dialogVisible4Edit"
      width="50%">
      <div>
        <span>分类名称：</span>
        <el-input class='cname' v-model="ecate.cat_name"></el-input>
      </div>
      <!-- <div>
        <span>父级分类：</span>
        <el-cascader
          :options="ecateList"
          v-model="eselectedOptions"
          :props='propsDefine'
          :show-all-levels="false">
        </el-cascader>
      </div> -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible4Edit = false">取 消</el-button>
        <el-button type="primary" @click="submitCate4Edit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import TreeGrid from './TreeGrid.vue'
import {getCategories, addCate, getCateById, editCate, deleteCate} from '../../api/api.js'
export default {
  data () {
    return {
      propsDefine: {
        value: 'cat_id',
        label: 'cat_name'
      },
      cateList: [],
      ecateList: [],
      selectedOptions: [],
      eselectedOptions: [],
      ecate: {
        cat_pid: '',
        cat_name: '',
        cat_level: ''
      },
      cate: {
        cat_pid: '',
        cat_name: '',
        cat_level: ''
      },
      dataSource: [],
      columns: [{
        text: '分类名称',
        dataIndex: 'cat_name',
        width: 500
      }, {
        text: '是否有效',
        dataIndex: 'cat_deleted',
        width: 100
      }, {
        text: '排序',
        dataIndex: 'cat_level',
        width: 100
      }],
      currentPage: 1,
      total: 10,
      pagesize: 5,
      dialogVisible4Add: false,
      dialogVisible4Edit: false
    }
  },
  methods: {
    submitCate4Edit () {
      // 编辑分类第二步
      // 编辑分类提交表单
      editCate(this.ecate).then(res => {
        if (res.meta.status === 200) {
          // 刷新列表
          this.initList()
          this.dialogVisible4Edit = false
          this.$message({
            type: 'success',
            message: res.meta.msg
          })
        }
      })
    },
    submitCate () {
      // 添加分类
      // 加工分类参数数据
      if (this.selectedOptions.length === 0) {
        this.cate.cat_pid = 0
        this.cate.cat_level = 1
      } else {
        this.cate.cat_pid = this.selectedOptions[this.selectedOptions.length - 1]
        // 设置层级
        this.cate.cat_level = this.selectedOptions.length === 1 ? 2 : 3
      }
      // 提交表单
      addCate(this.cate).then(res => {
        if (res.meta.status === 201) {
          // 刷新列表
          this.initList()
          // 关闭窗口
          this.dialogVisible4Add = false
          // 提示
          this.$message({
            type: 'success',
            message: res.meta.msg
          })
        }
      })
    },
    handleChange () {
      console.log(1)
    },
    addHandler () {
      // 获取所有的分类列表数据
      getCategories({type: 2}).then(res => {
        if (res.meta.status === 200) {
          this.cateList = res.data
          this.dialogVisible4Add = true
        }
      })
    },
    deleteCategory (cid) {
      // 调用删除分类接口
      deleteCate({id: cid}).then(res => {
        if (res.meta.status === 200) {
          // 刷新列表
          this.initList()
          this.$message({
            type: 'success',
            message: res.meta.msg
          })
        }
      })
    },
    showEditForm (cid) {
      // 编辑分类第一步
      // 获取分类下拉列表数据
      getCategories().then(res => {
        if (res.meta.status === 200) {
          this.ecateList = res.data
          // 获取数据后调用获取分类信息接口
          return getCateById({id: cid})
        }
      }).then(res => {
        if (res.meta.status === 200) {
          // 把数据填充到表单
          this.ecate.cat_pid = res.data.cat_id
          this.ecate.cat_name = res.data.cat_name
          this.ecate.cat_level = res.data.cat_level
          this.dialogVisible4Edit = true
        }
      })
    },
    refresh () {
      console.log('fresh')
    },
    handleSizeChange (val) {
      // 改变每页显示条数
      this.pagesize = val
      this.initList()
    },
    handleCurrentChange (val) {
      // 改变当前页码
      this.currentPage = val
      this.initList()
    },
    initList () {
      // 获取分类列表数据
      getCategories({type: 3, pagenum: this.currentPage, pagesize: this.pagesize}).then(res => {
        if (res.meta.status === 200) {
          this.dataSource = res.data.result
          this.pagesize = res.data.pagesize
          this.total = res.data.total
        }
      })
    }
  },
  components: {
    TreeGrid
  },
  mounted () {
    this.initList()
  }
}
</script>
<style scoped>
  .el-breadcrumb {
    background-color: #D3DCE6;
    height: 50px;
    line-height: 50px;
    padding-left: 10px;
  }
  .el-pagination {
    background-color: #D3DCE6;
    padding-top: 10px;
    height: 35px;
    line-height: 35px;
  }
  .cname {
    width: 300px;
  }
</style>
