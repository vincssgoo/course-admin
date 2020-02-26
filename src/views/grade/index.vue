<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="dialogVisible = true">新增</el-button>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter(value)">搜索</el-button>

      <el-input placeholder="请输入年级名称"
                v-model="listQuery.keyword"
                style="width: 320px;float:right;margin-right:20px"
                clearable>
      </el-input>

    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="155">
        <template slot-scope="scope">
          {{ scope.$index+1 }}
        </template>
      </el-table-column>

      <el-table-column align="center"
                       label="年级">
        <template slot-scope="scope">
          {{ scope.row.name }}
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       width="250"
                       align="center">
        <template slot-scope="scope"
                  style="">
          <div>
            <el-button @click="handleEdit(scope.row)"
                       type="primary">修改</el-button>
            <el-button @click="handleDelete(scope.row)"
                       type="danger">删除</el-button>
          </div>
          <div style="margin-top:8px">
            <el-button @click="goClass(scope.row)"
                       type="success">班级管理</el-button>
          </div>
        </template>
      </el-table-column>

    </el-table>
    <el-dialog :visible.sync="dialogVisible"
               :close-on-click-modal="false">
      <el-form ref="form"
               :model="form"
               label-position="left"
               label-width="70px">
        <el-form-item label="年级名称"
                      prop="name">
          <el-input v-model="form.name"
                    placeholder="请输入年级名称"
                    style="width:50%" />
        </el-form-item>
      </el-form>
      <div slot="footer"
           class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary"
                   @click="saveData"
                   :loading="btnLoading">确定</el-button>
      </div>
    </el-dialog>
    <!-- <router-view /> -->
  </div>
</template>
<script>
import request from "@/utils/request";

export default {
  data () {
    return {
      isSale: false,
      total: null,
      list: null,
      list_type: null,
      listLoading: true,
      btnLoading: false,
      dialogVisible: false,
      listQuery: {
        keyword: '',

      },
      form: {
        id: '',
        name: '',
        grade_id: "",
      }
    }
  },
  created () {
    this.getList();
  },
  watch: {
    dialogVisible (newVal, oldVal) {
      // 编辑框一异隐藏，马上清除旧数据
      if (newVal === false) {
        this.form = {
          id: '',
          name: '',
          grade_id: '',
        };
      }
    }
  },
  methods: {
    handleDelete (row) {
      this.$confirm("确定要删除该年级吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        request({
          url: "/api/backend/grade/delete",
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
    goClass (row) {
      this.$router.replace({
        path: '/class',
        query: {
          grade_id: row.id,
          // title: row.title,
          // school: row.course_site.name
        }
      })
    },
    getList () {
      this.listLoading = true;
      request({
        url: "/api/backend/grade/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.listLoading = false;
        console.log(this.list);

        // console.log(this.list);
        // this.listQuery.sale_status = ''
      });

    },
    handleEdit (item) {
      this.form = {
        id: item.id,
        name: item.name,
        grade_id: item.grade_id,
      };

      this.dialogVisible = true;
    },
    handleFilter (value) {
      this.listQuery.page = 1;
      this.getList();
    },
    saveData () {
      if (!this.form.name) {
        this.$message({
          type: "warning",
          message: "请输入班级名称"
        });
        return;
      }

      this.btnLoading = true;
      request({
        url: "/api/backend/grade/store",
        method: "post",
        data: this.form
      })
        .then(() => {
          this.btnLoading = false;
          this.dialogVisible = false;
          this.getList();
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        })
        .finally(() => {
          this.btnLoading = false;
        });
    },
  },


}

</script>

<style scope>
.user-avatar {
  width: 60px;
  height: 60px;
  border-radius: 6px;
}
</style>