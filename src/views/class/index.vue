<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="dialogVisible = true">新增</el-button>
      <el-button type="success"
                 plain
                 style="float:right;"
                 @click="backIndex">返回</el-button>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter(value)">搜索</el-button>

      <el-input placeholder="请输入年级/班级名称"
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
                       label="班级">
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

        </template>
      </el-table-column>

    </el-table>
    <el-dialog :visible.sync="dialogVisible"
               :close-on-click-modal="false">
      <el-form ref="form"
               :model="form"
               label-position="left"
               label-width="70px">
        <el-form-item label="班级名称"
                      prop="name">
          <el-input v-model="form.name"
                    placeholder="请输入班级名称"
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
      total: null,
      dialogVisible: false,
      schoolQuery: {
        keyword: '',
      },
      listQuery: {
        keyword: '',
        grade_id: '',
        page: 1,
        limit: 10,
      },
      form: {
        id: '',
        name: '',
        grade_id: '',
      },
      schoolList: null,
    }
  },
  created () {
    this.listQuery.grade_id = this.$route.query.grade_id

    this.getList();
    this.getSchoolList();
  },
  watch: {
    dialogVisible (newVal, oldVal) {
      // 编辑框一异隐藏，马上清除旧数据
      if (newVal === false) {
        this.form = {
          id: '',
          name: '',
          // grade_id: '',
        };
      }
    }
  },
  methods: {
    backIndex () {
      this.$router.replace({ path: '/grade/index' })
    },
    getSchoolList () {
      request({
        url: "/api/backend/grade/index",
        method: "get",
        params: this.schoolQuery
      }).then(response => {
        this.schoolList = response.data.data;
        console.log(this.schoolList);
      });

    },
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    handleDelete (row) {
      this.$confirm("确定要删除该班级吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        request({
          url: "/api/backend/classes/delete",
          method: "post",
          data: { id: row.id },
        }).then(() => {
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
    getList () {
      this.listLoading = true;
      request({
        url: "/api/backend/classes/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;
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
      this.form.grade_id = this.$route.query.grade_id
      this.btnLoading = true;
      request({
        url: "/api/backend/classes/store",
        method: "post",
        data: { name: this.form.name, grade_id: this.form.grade_id }
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