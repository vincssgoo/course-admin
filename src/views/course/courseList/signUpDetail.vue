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
      <el-button class="btns"
                 type="success"
                 style="float:right;margin-right:20px"
                 @click="postOrder">导出报名详情</el-button>

    </div>
    <div style="margin-bottom:15px;display:flex;flex-direction:row;">

      <span style="font-size:15px;font-weight:bold">{{school_name}}</span>
      <span style="font-size:15px;margin-left:30px;font-weight:bold">{{course_name}}</span>
      <span style="font-size:15px;margin-left:30px;">报名数量:{{total}}</span>

    </div>
    <el-table v-loading="listLoading"
              :data="list"
              element-loading-text="Loading"
              border
              fit
              highlight-current-row>
      <el-table-column align="center"
                       label="序号"
                       width="85">
        <template slot-scope="scope">
          {{ scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="头像"
                       width="120">
        <template slot-scope="scope">
          <img :src="scope.row.user.avatar"
               class="user-avatar">
        </template>
      </el-table-column>
      <el-table-column label="昵称"
                       width="140"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.user.nickname }}</span>
        </template>
      </el-table-column>
      <el-table-column label="联系电话"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.user.phone }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="姓名"
                       width="120"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.name}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="学校"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.school}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="班级"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.class}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="身份证号码"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.id_card_number}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="价格"
                       width="120">
        <template slot-scope="scope">
          <span>{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="报名时间">
        <template slot-scope="scope">
          <span>{{scope.row.user.school}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="班级"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.class}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="身份证号码"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.id_card_number}}</span>
        </template>
      </el-table-column>
    </el-table>

    <el-table v-loading="listLoading"
              element-loading-text="Loading"
              border=""
              fit
              highlight-current-row
              :data="exportExcels"
              id="rebateSetTable"
              style="display:none;">
      <el-table-column label="昵称"
                       width="140"
                       align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.user.nickname }}</span>
        </template>
      </el-table-column>
      <el-table-column label="联系电话"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          {{ scope.row.user.phone }}
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="姓名"
                       width="120"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.name}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="学校"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.school}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="班级"
                       width="180"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.class}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="status-col"
                       label="身份证号码"
                       width="200"
                       align="center">
        <template slot-scope="scope">
          <span>{{scope.row.user.id_card_number}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center"
                       label="价格"
                       width="120">
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
import FileSaver from 'file-saver'
import XLSX from 'xlsx'
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
      title: '123',
      course_name: '',
      school_name: '',
      exportExcels: [],
    }
  },
  created () {
    // this.title = this.$route.query.title
    console.log(this.$route.query.title);
    this.course_name = this.$route.query.title
    this.school_name = this.$route.query.school
    this.getList();

  },
  methods: {
    postOrder () {
      let listQuery = JSON.parse(JSON.stringify(this.listQuery));
      listQuery.page = 1;
      listQuery.limit = this.total;
      request({
        url: "/api/backend/course/enrollDetail",
        method: "get",
        params: listQuery
      }).then(response => {
        // this.exportExcels = response.data.data.map((item) => {
        //   item.order_no = "" + item.order_no
        //   return item
        // })
        this.exportExcels = response.data.data;
        this.$nextTick(function () {
          let xlsxParam = { raw: true };
          let wb = XLSX.utils.table_to_book(document.querySelector("#rebateSetTable"), xlsxParam);
          /* get binary string as output */
          let wbout = XLSX.write(wb, {
            bookType: "xlsx",
            bookSST: true,
            type: "array"
          });
          try {
            FileSaver.saveAs(
              new Blob([wbout], { type: "application/octet-stream" }),
              this.course_name + '.xlsx'
            );
          } catch (e) {
            if (typeof console !== "undefined") console.log(e, wbout);
          }
        });
      });
    },
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
        console.log(666);

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