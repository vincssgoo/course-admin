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
      <el-button type="primary"
                 plain
                 style="float:right;"
                 icon="el-icon-search"
                 @click="handleFilter">搜索</el-button>
    </div>
    <div style="margin-bottom:15px;">
      <span style="font-size:15px;">订单总额:{{amount}}</span>
    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="125">
        <template slot-scope="scope">
          {{ scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="订单号">
        <template slot-scope="scope">
          {{ scope.row.order_no }}
        </template>
      </el-table-column>
      <el-table-column label="昵称"
                       width="150"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.user.nickname }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="头像"
                       width="150">
        <template slot-scope="scope">
          <img :src="scope.row.user.avatar"
               class="user-avatar">
        </template>
      </el-table-column>
      <el-table-column label="联系电话"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.user.phone }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="课程名称"
                       width="200">
        <template slot-scope="scope">
          {{ scope.row.course.title }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="价格"
                       width="150">
        <template slot-scope="scope">
          {{ scope.row.price }}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="购买时间">
        <template slot-scope="scope">
          {{ scope.row.pay_datetime }}
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

export default {
  data () {
    return {
      list: null,
      amount: null,
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
    this.getList()
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
      this.listLoading = true;
      request({
        url: "/api/backend/finance/orderFlow",
        method: "get",
        params: this.listQuery
      }).then(response => {
        this.list = response.data.data;
        this.total = response.data.total;
        this.amount = response.data.order_amount;
        this.listLoading = false;
      });

    },
  }
}
</script>
