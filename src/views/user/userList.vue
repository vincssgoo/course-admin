<template>
  <div class="app-container">
    <div class="block"
         style="margin-bottom:15px;">

      <span style="margin-left:40px;">时间段</span>
      <el-date-picker style="margin-left:20px;"
                      v-model="listQuery.start_datetime"
                      type="date"
                      placeholder="开始日期时间"
                      value-format="yyyy-MM-dd"></el-date-picker>
      至
      <el-date-picker v-model="listQuery.end_datetime"
                      type="date"
                      placeholder="结束日期时间"
                      value-format="yyyy-MM-dd">
      </el-date-picker>
      <el-input placeholder="请输入用户名/电话"
                v-model="listQuery.keyword"
                style="width: 320px;"
                clearable>
      </el-input>
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter">搜索</el-button>
    </div>
    <div style="margin-bottom:15px;">
      <span style="font-size:15px;">用户数量:{{total}}</span>
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
      <el-table-column align="center"
                       label="头像"
                       width="110">
        <template slot-scope="scope">
          <!-- {{ scope.row.avatar }} -->
          <img :src="scope.row.avatar"
               class="user-avatar">
          <!-- <upload-one v-model="scope.row.avatar"></upload-one> -->
        </template>
      </el-table-column>
      <el-table-column label="昵称"
                       width="110"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.nickname }}</span>
        </template>
      </el-table-column>
      <el-table-column label="联系电话"
                       width="130"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.phone }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="姓名"
                       width="120"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.name}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="学校"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.school}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="班级"
                       width="160"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.class}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="身份证号码"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.id_card_number}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="注册时间"
                       width="200">
        <template slot-scope="scope">
          <span>{{ scope.row.created_at }}</span>
        </template>
      </el-table-column>
    </el-table>
    <div class="block"
         style="margin-top:25px">
      <!-- <span class="demonstration">完整功能</span> -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="listQuery.page"
                     :page-sizes="[5,10,20,30, 50]"
                     :page-size="listQuery.limit"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import request from "@/utils/request";
// import { mapGetters } from "vuex";

export default {
  // filters: {
  //   statusFilter (status) {
  //     const statusMap = {
  //       published: 'success',
  //       draft: 'gray',
  //       deleted: 'danger'
  //     }
  //     return statusMap[status]
  //   }
  // },
  data () {
    return {
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        keyword: '',
        end_datetime: '',
        start_datetime: '',
      },

    }
  },
  created () {
    this.getList();
  },
  methods: {
    handleFilter () {
      this.listQuery.page = 1;
      this.getList();
    },
    handleSizeChange (val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange (val) {
      this.listQuery.page = val;
      this.getList();
    },
    getList () {
      console.log(123);
      // console.log(this.dateArr);
      this.listLoading = true;
      // this.listQuery.start_datetime = this.listQuery.start_datetime ? this.listQuery.start_datetime + ' 00:00:00 ' : '';
      // this.listQuery.end_datetime = this.listQuery.end_datetime ? this.listQuery.end_datetime + ' 23:59:59 ' : '';

      request({
        url: "/api/backend/user/index",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;
        console.log(this.list);
        console.log(this.total);


      });

    },
  }
}
</script>
<style scope>
.user-avatar {
  width: 60px;
  height: 60px;
  border-radius: 6px;
}
</style>