<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="goModified">添加</el-button>

    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="95">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>
      <el-table-column label="问题"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column label="答案"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.content }}
        </template>
      </el-table-column>
      <el-table-column label="权重"
                       align="center"
                       width="85">
        <template slot-scope="scope">
          {{ scope.row.weight }}
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       width="230"
                       align="center">
        <template slot-scope="scope">
          <el-button type="primary"
                     @click="handleEdit(scope.row)">修改</el-button>
          <el-button type="danger"
                     @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>

    </el-table>

    <div class="block"
         style="margin-top:25px">
      <!-- <span class="demonstration">完整功能</span> -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="listQuery.page"
                     :page-sizes="[5, 10, 20, 50,100]"
                     :page-size="listQuery.limit"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import request from "@/utils/request";

export default {
  data () {
    return {
      total: null,
      list: null,
      listLoading: true,
      btnLoading: false,
      listQuery: {
        page: 1,
        limit: 10,
      },
      form: {
        id: "",
        name: "",
      },
    }
  },
  created () {
    this.getList();
  },
  methods: {
    getList () {

      this.listLoading = true;
      request({
        url: "/api/backend/question/index",
        method: "get",
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;

      });
    },
    goModified () {
      this.$router.replace({ path: '/questionModified' })

    },
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    handleEdit (item) {
      this.$router.replace({        path: '/questionModified',
        query: { id: item.id }      })
    },
    handleDelete (row) {
      this.$confirm("确定要删除问题吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        console.log(1234);
        request({
          url: "/api/backend/question/delete",
          method: "post",
          data: { id: row.id },
        }).then(() => {
          console.log(123);

          // 删除最后一条数据时无数据问题
          this.list.length <= 1 ? this.listQuery.page-- : false;
          this.getList();
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        });
      });
    },

    // saveData () {
    //   if (!this.form.name) {
    //     this.$message({
    //       type: "warning",
    //       message: "请输入学校分点名称"
    //     });
    //     return;
    //   }

    //   this.btnLoading = true;
    //   request({
    //     url: "/api/backend/courseSite/store",
    //     method: "post",
    //     data: this.form
    //   })
    //     .then(() => {
    //       this.btnLoading = false;
    //       this.dialogVisible = false;
    //       this.getList();
    //       this.$message({
    //         type: "success",
    //         message: "操作成功!"
    //       });
    //     })
    //     .finally(() => {
    //       this.btnLoading = false;
    //     });
    // },
  },


}

</script>
