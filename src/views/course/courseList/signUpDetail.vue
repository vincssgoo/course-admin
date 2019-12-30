<template>
  <div class="app-container">
    <div class="block"
         style="margin-bottom:15px;">
      <span class="demonstration"
            style="font-size:15px;">报名时间 </span>
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
      <span style="font-size:15px;">报名户数量:{{total}}</span>
    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="195">
        <template slot-scope="scope">
          {{ scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="头像"
                       width="200">
        <template slot-scope="scope">
          <img :src="scope.row.user.avatar"
               class="user-avatar">
        </template>
      </el-table-column>
      <el-table-column label="昵称"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.user.nickname }}</span>
        </template>
      </el-table-column>
      <el-table-column label="联系电话"
                       width="220"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.user.phone }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="姓名"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.name}}</span>
        </template>
      </el-table-column>

      <el-table-column align="center"
                       label="价格"
                       width="220">
        <template slot-scope="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="报名时间">
        <template slot-scope="scope">
          <span>{{ scope.row.pay_datetime }}</span>
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
        id: "",
        start_datetime: '',
        end_datetime: '',
        keyword: '',
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
      this.listQuery.id = this.$route.query.id;
      this.listLoading = true;
      request({
        url: "/api/backend/course/enrollDetail",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.listLoading = false;
        console.log(this.list);

      });
    },
  }
}
</script>
<style scope>
.user-avatar {
  width: 90px;
  height: 90px;
  border-radius: 6px;
}
</style>