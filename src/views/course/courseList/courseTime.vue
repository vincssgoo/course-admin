<template>
  <div class="app-container">
    <div style="margin-bottom:15px;">
      <el-button type="primary"
                 @click="dialogVisible = true">新增</el-button>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter">搜索</el-button>
      <div style="float:right">
        <span class="demonstration"
              style="font-size:15px;">报名时间 </span>
        <el-date-picker style="margin-left:20px;"
                        v-model="listQuery.start_date"
                        type="date"
                        placeholder="开始日期时间"
                        value-format="yyyy-MM-dd"></el-date-picker>
        至
        <el-date-picker v-model="listQuery.end_date"
                        type="date"
                        placeholder="结束日期时间"
                        value-format="yyyy-MM-dd">
        </el-date-picker>
      </div>

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
      <el-table-column label="时间"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.date }}
        </template>
      </el-table-column>
      <el-table-column label="操作"
                       width="230"
                       align="center">
        <template slot-scope="scope">
          <el-button type="danger"
                     @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>

    </el-table>

    <el-dialog :visible.sync="dialogVisible"
               width="30%">
      <el-form>
        <el-form-item label="时间">

          <el-date-picker v-model="form.date"
                          type="date"
                          placeholder="选择日期">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="saveData"
                   :loading="btnLoading">提 交</el-button>
      </span>
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
      dialogVisible: false,
      listQuery: {
        page: 1,
        limit: 10,
        course_id: '',
        start_date: '',
        end_date: '',
      },
      form: {
        date: '',
      },
      year: '',
      month: '',
      day: '',
      Date: '',
    }
  },
  created () {
    this.getList()
  },
  methods: {
    getList () {
      this.listQuery.course_id = this.$route.query.course_id;
      this.listLoading = true;
      request({
        url: "/api/backend/courseDate/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.total = response.data.total;

        this.list = response.data.data;

        this.listLoading = false;
      });
    },
    handleFilter () {
      this.listQuery.page = 1;
      this.getList();
    },
    handleDelete (row) {
      this.$confirm("确定要删除时间吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        request({
          url: "/api/backend/courseDate/delete",
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
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    saveData () {
      let test = new Date(this.form.date)
      this.year = new Date(this.form.date).getFullYear()
      this.month = new Date(this.form.date).getMonth() + 1
      this.day = new Date(this.form.date).getDate()
      this.Date = this.year + '-' + this.month + '-' + this.day
      this.btnLoading = true;
      request({
        url: "/api/backend/course/setCourseDate",
        method: "post",
        data: {
          date: this.Date,
          course_id: this.listQuery.course_id
        }
      }).then(() => {
        this.getList()
        this.$message({
          type: "success",
          message: "操作成功!"
        });
      })
        .finally(() => {
          this.btnLoading = false;
        });
    }


  }
}
</script>
